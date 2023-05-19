<!-- PROJECT LOGO -->
<br />
<div align="center">
  <img src="https://github.com/DemikFR/Data-Pipeline-Python-Azure-Data-Lake/assets/102700735/e8edbe09-c076-44e4-b927-9a649e81db55">
  <h1 align="center">Pipeline de Extração de Dados da Web para o Azure Data Lake</h1>

  <p align="center">
    Ingestão de dados no Azure Data Lake com web scraping no Python
  </p>
  <p align="center">
    Os dados usados se encontram na base de <a href="https://www.conab.gov.br/info-agro/safras">dados abertos da CONAB</a>.
  </p>
</div>



<!-- TABLE OF CONTENTS -->
<details>
  <summary>Table of Contents</summary>
  <ol>
    <li>
      <a href="#sobre-o-projeto">Sobre o Projeto</a>
      <ul>
        <li><a href="#ferramentas">Ferramentas</a></li>
      </ul>
    </li>
    <li>
      <a href="#iniciar-o-projeto">Iniciar o Projeto</a>
      <ul>
        <li><a href="#preparação-do-azure">Preparação do Azure</a></li>  
      </ul>
     </li>
    <li><a href="#extração-dos-dados-web-scraping">Extração dos Dados</a></li>
    <li><a href="#ingestão-dos-dados">Ingestão dos Dados</a></li>
    <li><a href="#agradecimentos">Agradecimentos</a></li>
    <li><a href="#license">License</a></li>
    <li><a href="#contact">Contact</a></li>
  </ol>
</details>



<!-- Sobre o Projeto -->
## Sobre o Projeto

O projeto foi originado por um desafio de minha graduação de banco de dados na FIAP (Faculdade de Informática e Administração), que consiste em pensar em um problema do agronegócio para ser resolvido utilizando uma plataforma de data lake.

Para ser realizada o problema, foi escolhido usar o processo de ELT e o Azure Data Lake Gen1, pois além de realizar a análise do momento, os dados deverão ser reaproveitados no futuro.


### Ferramentas

Para realizar este projeto, foi usado as seguintes ferramenta:


* [![Python][Python.py]][Python-url]
* [![Microsoft Azure][Azure.azr]][Azure-url]



<!-- Iniciar o Projeto -->
## Iniciar o Projeto

1. Clone este Repositório
   ```sh
   git clone https://github.com/DemikFR/Data-Pipeline-Python-Azure-Data-Lake.git
   ```
2. Instale todas as bibliotecas em seu Python
   ```py
    pip install wget
    pip install azure-mgmt-resource
    pip install azure-mgmt-datalake-store
    pip install azure-datalake-store
   ```
3. Verifique a sua credencial <i>Tenant</i> ID do Azure, conforme a <a href="https://learn.microsoft.com/pt-br/azure/active-directory/fundamentals/active-directory-how-to-find-tenant">documentação da Microsoft.</a> e depois coloque ela no seu código que se encontra no script
   ```py
    adlCreds = lib.auth(tenant_id='YOUR_TENANT_ID', resource = 'https://datalake.azure.net/')
   ```


### Preparação do Azure

Para criar o Azure Data Lake Gen1 é importante fazer antes um grupo de recursos do Azure. É fundamental observar que o nome do grupo de recursos não deve conter caracteres especiais, incluindo espaços, e deve ser composto apenas por letras minúsculas. Essa regra de nomenclatura é essencial para garantir a correta configuração e funcionamento do Azure Data Lake Gen1.

![Grupo de Recursos do Azure](https://github.com/DemikFR/Data-Pipeline-Python-Azure-Data-Lake/assets/102700735/fd60b0fb-6e86-424f-a002-98074dcac679)


Com o grupo de recursos será possível organizar os serviços que tem um mesmo objetivo, neste caso, para analisar as safras de produtos agrícolas brasileiro.

Agora que o Grupo de Recursos foi criado, podemos prosseguir com a criação do Data Lake. Selecione o recurso <i>Data Lake Storage Gen1</i> e, em seguida, forneça as informações necessárias. É importante observar o nome da instância que você está criando, pois esse nome será utilizado para acessar o Data Lake por meio do Python. Certifique-se de escolher um nome significativo e memorável para facilitar o acesso e o gerenciamento do Data Lake posteriormente.

![Criação do Azure Data Lake Storage Gen1](https://github.com/DemikFR/Data-Pipeline-Python-Azure-Data-Lake/assets/102700735/a4f11faa-c1b3-4416-a93b-ef2d3ab85625)


Após concluir a criação do Azure Data Lake Gen1, já será possível começar a utilizá-lo.

![Interface Azure Data Lake](https://github.com/DemikFR/Data-Pipeline-Python-Azure-Data-Lake/assets/102700735/94dfae99-5631-4aff-a38d-a2a817cb1837)




## Extração dos Dados (Web Scraping)

Após analisar a forma de disponibilização dos dados, foi verificado que as bases estão armazenadas em formatos mistos, sendo tanto em .XLS quanto em .XLSX. Além disso, antes de acessar a página de download dos dados, é necessário passar por outra página que os separa em seis categorias: <b>Safra de Grãos</b>, <b>Safra de Café</b>, <b>Safra de Cana-de-Açúcar</b>, <b>Séries Históricas</b>, <b>Progresso de Safra</b> e <b>Mapeamentos Agrícolas</b>. Inicialmente, será coletar apenas os dados das quatro primeiras categorias, nos formatos .XLSX.

Com a biblioteca Beautiful Soup, será possível realizar o processo de web scraping que automatizará o download e envio dos arquivos de dados ao Azure Data Lake. 

Assim, foi criada uma função que recebe uma URL, no caso a página que dá acesso às categorias dos dados e uma expressão expressão de busca que será aplicada ao atributo "title" do HTML, a fim de encontrar algo semelhante.

   ```py
    def get_soup(url, search):
    response = requests.get(url)
    
    # Verificar se a conexão foi bem sucedida
    if response.status_code == 200:
        soup = bs(response.content, 'html.parser')

        # Retornar as páginas onde terão as de download
        return soup.find_all('a', href=True, attrs={'title': re.compile(search)}) 
    else:
        return f'Página{link["title"]} está inacessível. Código: {response.status_code}'
   ```

Após a criação da função, foi implementado um loop "for" que percorre a lista de elementos retornados pela função. Em seguida, outro loop é utilizado para realizar uma nova busca por elementos dentro da página acessada com base na primeira lista. Dessa forma, o link de acesso ao dataset é extraído e será posteriormente inserido na função de ingestão de dados, que será detalhada no capítulo dedicado a essa funcionalidade.

   ```py
    url_base = 'https://www.conab.gov.br'

    url_acesso = 'https://www.conab.gov.br/info-agro/safras'

    lista_repo = get_soup(url_acesso, 'Safra')[0:3]

    for i in lista_repo:
    
      # Buscar datasets em cada repositório de dados do Conab
      link_repo = url_base+i['href'] # Funcionalidade do Python que ajuda a acessar elementos específicos do HTML

      for j in get_soup(link_repo, 'xlsx'):

          # Acessar os dados de cada dataset e inserir no Azure Data Lake
          link_dataset = url_base+j['href'] # Funcionalidade do Python que ajuda a acessar elementos específicos do HTML

          ingest(adlsFileSystemClient, j['title'], link_dataset)
   ```



## Ingestão dos Dados

No momento em que a extração dos dados foi concluída com sucesso, já é possível prosseguir com o processo de ingestão. No entanto, antes de iniciar, é fundamental configurar o ambiente que estabelecerá a conexão com o Azure Data Lake. Após realizar a etapa 3 do <a href="#iniciar-o-projeto">Iniciar o Projeto</a>, deve-se conectar no seu ADL já criado na etapa de <a href="#preparação-do-azure">Preparação do Azure</a> e depois com o código abaixo, usar o nome do ADLGen1.
   ```py
    adlsAccountName = 'datalakeminsaitsafras'
    adlsFileSystemClient = core.AzureDLFileSystem(adlCreds, store_name=adlsAccountName)
   ```
   
A função de ingestão é projetada para receber três parâmetros essenciais. O primeiro parâmetro é o "file_system", que representa o sistema de arquivos atuando como um contêiner no Azure Data Lake (ADL). Como mencionado anteriormente, esse sistema de arquivos foi criado utilizando o código fornecido anteriormente.

O segundo parâmetro é o "file_dl_name", que define o nome do arquivo no Azure Data Lake. Esse parâmetro é crucial para identificar o arquivo específico que será armazenado no Data Lake.

Por fim, o terceiro parâmetro é o "link_file", que no contexto desse pipeline é o link de download do dataset. É por meio desse caminho que a função de ingestão será capaz de localizar o arquivo a ser importado e realizar a operação de ingestão no Azure Data Lake.

Note que em primeiro momento, será utilizado a biblioteca wget para realizar o download do dataset na pasta e depois subir o arquivo na nuvem, isso manterá a integridade dos dados.

   ```py
    def ingest(file_system, file_dl_name, link_file):

      try:
      # Download the Dataset
        response = wget.download(link_file) # Baixar o dataset

        ## Upload a file

        multithread.ADLUploader(file_system, lpath=f'/content/{file_dl_name}', rpath=f'/Datasets_Safras/{file_dl_name}'
                                , nthreads=64, overwrite=True) # Pega o arquivo baixado e realiza o upload para o Azure
        print(f'O upload do arquivo {file_dl_name} foi efetuado com sucesso!')
      except exception:
        print(f'Erro! O upload do arquivo {file_dl_name} não foi bem sucedido!') # Em caso de algum erro ocorrer ao baixar ou enviar o dataset para o Azure
   ```
   
Por fim, os dados já estarão presentes no Azure Data Lake, pronto para os próximos procedimentos de análise.

![Datasets no ADL](https://github.com/DemikFR/Data-Pipeline-Python-Azure-Data-Lake/assets/102700735/d5e327b0-a8b8-4404-8977-6da95705b189)



## Agradecimentos

O trabalho está sendo feito junto com a minha amiga Yuki Shimura, que ficou encarregada da transformação e análise dos dados.


<!-- LICENSE -->
## License

Distributed under the MIT-LICENSE. See `LICENSE.txt` for more information.



<!-- CONTACT -->
## Contact

Demik Freitas - [Linkedin](https://www.linkedin.com/in/demik-freitas/) - demik.freitast2d18@gmail.com

Project Link: [https://github.com/DemikFR/Data-Pipeline-Python-Azure-Data-Lake](https://github.com/DemikFR/Data-Pipeline-Python-Azure-Data-Lake)



<!-- MARKDOWN LINKS & IMAGES -->
<!-- https://www.markdownguide.org/basic-syntax/#reference-style-links -->
[Python.py]: https://img.shields.io/badge/python-3670A0?style=for-the-badge&logo=python&logoColor=ffdd54
[Python-url]: https://www.python.org/
[Azure.azr]: https://img.shields.io/badge/azure-%230072C6.svg?style=for-the-badge&logo=microsoftazure&logoColor=white
[Azure-url]: https://azure.microsoft.com/pt-br/free/search/?ef_id=_k_CjwKCAjw9pGjBhB-EiwAa5jl3AzQsgrvNjfAVF_2lmRSWukaiU7bsXh-UG1YxqLMh4DcsKz0YrhwLhoC7_UQAvD_BwE_k_&OCID=AIDcmmzmnb0182_SEM__k_CjwKCAjw9pGjBhB-EiwAa5jl3AzQsgrvNjfAVF_2lmRSWukaiU7bsXh-UG1YxqLMh4DcsKz0YrhwLhoC7_UQAvD_BwE_k_&gclid=CjwKCAjw9pGjBhB-EiwAa5jl3AzQsgrvNjfAVF_2lmRSWukaiU7bsXh-UG1YxqLMh4DcsKz0YrhwLhoC7_UQAvD_BwE
