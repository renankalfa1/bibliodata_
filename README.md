## Registros Bibliográficos Digitais - Investigação

Esta atividade foi motivada pelo seguinte comentário feito no SNCat:

![image](https://github.com/user-attachments/assets/d5fa0b7e-7f0b-4925-9451-7795ff96f38d)

Com isso, eu pensei nas seguintes alternativas:
1. Com base no nome ou no código do registro/publicação seriada, pesquisar em algum site e verificar o meio (impresso ou digital).
2. Encontrar uma base de dados que contenha as publicadas em meio digital.
3. Pesquisar o nome no google para verificar se existem pdfs.

### 1. Alternativa 1 - Sites Externos

Durante a minha pesquisa, encontrei o site do (portal ISSN)[https://portal.issn.org/#services]. Nele é possível pesquisar por meio do código ISSN ou pelo título. Com isso, pelos dados do Bibliodata, fiz o teste abaixo:

![image](https://github.com/user-attachments/assets/a7a3c6fe-5123-4af6-b19f-984264a88f60)

Note que no site informa o "meio", que no caso diz que é impresso. Com isso, veio na minha cabeça a seguinte pergunta: Se um registro passar para o meio digital, o site irá apontar isso?.

Obs: na descrição do site afirma que contém registros bibliográficos impressos e on-line (https://portal.issn.org/#services).

#### 1.1 Verificações manuais.

Como no banco do bibliodata contém um pouco mais de 30.000 registros, selecionei 15 registros para essa verificação manual por meio da seguinte consulta SQL:
```sql
select issn, titulo, * from registro_bibliografico
where issn is not null 
order by id
limit 15;
```

Por sorte, ao pesquisar pelo registro de ISSN "10910808", obtive o seguinte resultado:

![image](https://github.com/user-attachments/assets/d746d676-d37f-4c47-96d1-417a3e383094)

Acredito que este seja um caso de registro que posteriormente foi publicado de maneira digital, já que não constam muitas informações no site acerca dos registros, como é evidenciado na imagem abaixo:

![image](https://github.com/user-attachments/assets/7f574e3d-dd54-438e-b078-b44c69f03421)

#### 1.2 Verificação Automatizada

Com base no resultado da verificação manual, planejei criar um script Python que seguirá os seguintes passos:

1. Conecta-se ao banco de dados.
2. Realiza a consulta retornando o campo ISSN de todos os registros bibliográficos que o contenham.
3. Cria um dataframe de resultados (campos: ISSN, Resultado, ISSN_Associados) para armazenar os resultados da pesquisa.
Para cada código ISSN,
4. No site "https://portal.issn.org/", consulta o código em questão
5. Caso não haja resultados, armazenar "Sem resultados" no dataframe de resultados.
6. Caso entre na página do ISSN, verificar o meio de publicação. Caso seja digital, armazenar "Único - Digital", caso contrato, "Único - Impresso".
7. Caso retorne mais de um resultado, armazenar os ISSN no dataframe delimitados por vírgula e também armazenar "Vários - Digital".
