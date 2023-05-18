<!-- PROJECT LOGO -->
<br />
<div align="center">
  <img src="https://github.com/DemikFR/Data-Pipeline-Python-Azure-Data-Lake/assets/102700735/9656d571-e6a0-439d-93d5-4ba537f0f523"/>
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
    <li><a href="#iniciar-o-projeto">Iniciar o Projeto</a></li>
    <li><a href="#extração-dos-dados-web-scraping">Extração dos Dados</a></li>
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
2. Instale todas as bibliotecas em seu Jupyter ou outra IDE usada
   ```py
    pip install wget
    pip install azure-mgmt-resource
    pip install azure-mgmt-datalake-store
    pip install azure-datalake-store
   ```
3. Verifique a sua credencial <i>Tenant</i> ID do Azure necessárias, conforme a <a href="https://learn.microsoft.com/pt-br/azure/active-directory/fundamentals/active-directory-how-to-find-tenant">documentação da Microsoft.</a> e depois coloca-la no código que se encontra no script
   ```py
    adlCreds = lib.auth(tenant_id='YOUR_TENANT_ID', resource = 'https://datalake.azure.net/')
   ```



## Extração dos Dados (Web Scraping)

Após estudar como os dados estão disponibilizados, foi visto que as bases estão em Excel, mesclando entre o .XLS e .XLSX, além disso, antes de acessar a pagína em que os dados podem ser baixados, existe uma outra página em que separa os datasets em 6 categorias que são <b>Safra de Grãos</b>, <b>Safra de Café</b>, <b>Safra de Cana-de-Açucar</b>, <b>Séries Históricas</b>, <b>Progresso de Safra</b> e <b>Mapeamentos Agrícolas</b>. Em primeiro momento só serão coletados os dados dos 4 primeiros e arquivos XLSX. 

Com a biblioteca Beautiful Soup, será possível realizar o processo de web scraping que automatizará o download e envio dos arquivos de dados ao Azure Data Lake. 





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
