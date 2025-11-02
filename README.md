# Desafio Técnico: Engenharia de Dados - SiCooperative

Olá!

Este repositório contém a minha solução para o desafio técnico de Engenharia de Dados. O objetivo era criar um pipeline de ETL para a "SiCooperative LTDA", focado em resolver um problema de negócio real: a dificuldade em centralizar informações de diferentes fontes para agilizar a tomada de decisão.

## As ferramentas que eu escolhi

* **Banco de Dados:** PostgreSQL
* **Linguagem:** Python 3
* **Processamento/ETL:** Apache Spark (via PySpark)
* **Ambiente/Automação:** Docker & Docker Compose

## Minhas Escolhas Tecnológicas (O porquê)

Eu quis abordar o desafio com ferramentas que não apenas resolvessem o problema de hoje, mas que estivessem prontas para crescer.

### PostgreSQL
Optei pelo PostgreSQL para simular o banco de dados de origem. É uma escolha robusta, de código aberto e, como solicitado, uma das tecnologias padrão de mercado para SGBDs. Ele simula perfeitamente o ambiente transacional de onde os dados seriam extraídos.

### PySpark (Apache Spark)
Para o ETL, o PySpark foi a escolha natural. O desafio pedia um framework de processamento distribuído, e o Spark é o líder nesse quesito. Mesmo que a massa de dados inicial seja pequena, essa arquitetura já nasce pronta para escalar para *milhões* de movimentações sem precisar redesenhar o core do processo.

## Como Executar o Projeto (Tudo automatizado!)

Para facilitar sua avaliação, todo o ambiente é 100% automatizado com Docker. Você não precisa instalar Python, Spark ou PostgreSQL na sua máquina. Apenas o Docker.

### Pré-requisitos
* Docker
* Docker Compose

### Passos para Execução

1.  **Clone o Repositório:**
    ```bash
    git clone [URL_DO_SEU_REPOSITORIO]
    cd [NOME_DO_PROJETO]
    ```

2.  **Configure as Variáveis de Ambiente:**
    Crie um arquivo chamado `.env` na raiz do projeto. (Este arquivo é ignorado pelo Git, então seus segredos estão seguros).
    ```ini
    # .env
    POSTGRESS_USER=postgres
    POSTGRESS_PASSWORD=mysecretpassword
    POSTGRESS_DB=postgres
    ```

3.  **Execute o Docker Compose:**
    Este comando vai construir as imagens e iniciar os containers na ordem correta:
    ```bash
    docker-compose up --build
    ```

### O que acontece quando você roda o comando?
1.  O Docker Compose inicia o banco de dados `postgres-db`.
2.  O banco, ao iniciar, executa automaticamente os scripts da pasta `/sql` para **criar as tabelas** e **inserir os dados fictícios**.
3.  Só então, o serviço `spark-etl-job` começa.
4.  O script PySpark `etl_job.py` é executado. Ele se conecta ao banco (pela rede interna), extrai os dados, faz os joins necessários e transforma tudo.
5.  No final, você verá uma nova pasta `./output` aparecer na raiz do seu projeto, contendo o arquivo `.csv` unificado!

## Pontos de Atenção e Decisões de Modelagem

O ponto que exigiu mais interpretação foi a regra de negócio para a junção das tabelas.

* **O Dilema:** O desafio informa que "Um associado pode ter vários cartões e várias contas". O `movimento` se liga ao `cartao`, e tanto o `cartao` quanto a `conta` se ligam ao `associado`. Não há uma ligação direta entre o movimento e a conta.

* **Minha Decisão:** Para criar o arquivo flat solicitado, optei por juntar o movimento (via cartão) com *todas* as contas do respectivo associado.
    * **Implicação:** Se um associado tem 2 contas, cada movimento de seu cartão aparecerá 2 vezes no arquivo final (uma para cada conta).
    * **Justificativa:** Esta foi a interpretação que me pareceu mais alinhada com a necessidade de "achatar" os dados. Em um cenário real, este seria um ótimo ponto para validar com a equipe de negócios para confirmar se essa é a regra desejada.

## O que eu faria com mais tempo (Próximos Passos)

Esta solução atende 100% aos requisitos do desafio. Se este projeto fosse evoluir para produção, estes seriam os próximos passos lógicos que eu tomaria:

1.  **Mudar para Parquet:** O CSV é ótimo para visualização, mas para um Data Lake real, eu migraria o formato de saída para **Parquet**. É um formato colunar, muito mais performático para consultas analíticas e ocupa menos espaço.

2.  **Particionar os Dados:** Em vez de um arquivo único, eu particionaria os dados de saída (por exemplo, por data do movimento: `.../ano=2025/mes=11/...`). Isso acelera *absurdamente* as consultas futuras, pois o motor de busca (como o próprio Spark ou o Power BI) pode pular pastas inteiras que não lhe interessam.

3.  **Orquestração com Airflow:** O Docker Compose é perfeito para rodar o job uma vez. Para agendamento (ex: rodar toda noite), monitoramento de falhas e retentativas, eu migraria essa lógica para uma DAG no **Apache Airflow**.

4.  **Testes Unitários:** Eu implementaria os testes bônus! Usaria `pytest` e `pyspark` para criar pequenos DataFrames de teste e validar que as funções de transformação (especialmente os joins) estão se comportando exatamente como o esperado em todos os cenários.

5. **Uma Abordagem Séria sobre Segurança**

Desde o início, uma das minhas maiores preocupações foi não expor dados sensíveis, como senhas de banco de dados, no repositório. Em um ambiente real, isso é crítico.

* **Proteção de Credenciais:** Todas as credenciais (usuário, senha, nome do banco) são gerenciadas por um arquivo `.env`. Este arquivo **não** é enviado ao Git, garantindo que nenhum segredo seja vazado.

* **Isolamento de Rede:** Você notará que o `docker-compose.yml` não expõe a porta do PostgreSQL para o "mundo exterior". O container do Spark se comunica com o banco através da rede interna privada do Docker. É uma prática muito mais segura que deixa o banco de dados isolado.

Obrigado pela oportunidade!
