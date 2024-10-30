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

![webscraper](https://github.com/user-attachments/assets/9df9c3b6-fbe5-4e2e-b60e-c7afb85c0cb5)
Estatísticas:
- Tempo para verificação de cada registro: 50s
- Quantidade de registros: 27.532
- Tempo estimado: 382h
- Tempo estimado (multi - 40 rôbos): 9h30

Obs: como conseguimos acesso a lista completa dos ISSN do Brasil, esta alternativa foi concluída.

#### 1.3 Lista Completa dos ISSN - Brasil

Como conseguimos acesso a uma base dos ISSN do Brasil, esta verificação se tornou mais fácil. Porém, na base em questão há uma limitação de conter apenas registros do Brasil.

Para realizar a verificação, precisamos pesquisar o ISSN desejado na variável "ISSN-L" (Linking ISSN), como é evidenciado na imagem abaixo:

![image](https://github.com/user-attachments/assets/e8889097-80ad-426a-9e49-0a9cb87fe4d4)

Na sprint passada, ao conseguir levantar uma lista de ISSN válidos, eu realizei um filtro de forma a eliminar os registros que contenham "X", o que na verdade foi um erro pois existem sim registros bibliográficos como no exemplo abaixo:

![image](https://github.com/user-attachments/assets/49135bf3-8be7-4641-9a3e-3c4ea7d547d4)

Tendo em mãos a lista dos registros bibliográficos do Bibliodata que continham ISSN e da lista completa de ISSN do Brasil, segui o seguintes passos:
1. Criei um arquivo Excel com quatro abas: ISSN Brasil, ISSN Brasil - Online, Bibliodata e Resumo.
2. Na aba "Bibliodata", retirei os seguintes caractéres: "-", "." e "/".
3. Na aba "ISSN Brasil - Online", retirei o hífen.
4. Na aba "Bibliodata", realizei um CONT.SE > 0 de maneira a verificar se dado ISSN existe na aba "ISSN Brasil - Online".

Limitações:
1. Somente registros bibliográficos que possuem ISSN e são do Brasil.
2. Não contempla ISSN cancelados que foram "transferidos" para um outro código (imagem abaixo).

![image](https://github.com/user-attachments/assets/f6a9d62b-5afa-444a-b1f7-e837a5d8ed89)


### Resultados

|         Possui   versão digital?         |                      Quantidade de registros                     |              %             |
|:----------------------------------------:|:----------------------------------------------------------------:|:--------------------------:|
| Não possui                               |                                                          19.496  | 65,27%                     |
| Possui                                   |                                                          10.376  | 34,73%                     |

Obs: o valor real será maior do que o encontrado, tendo em vista que não foi possível delimitar para somente registros bibliográficos que contenham ISSN e são do Brasil.

#### Atualização - 30/10/2024

Como base na lista de ISSN do Brasil, consegui filtrar somente registros do Brasil e abaixo encontra-se o seu comportamento:

| Possui versão   digital?                 | Quantidade de registros | %                          |
|------------------------------------------|-------------------------|----------------------------|
| Não possui                               | 9010                    | 47,24%                     | Desenvolver script, alterar a tabela de registros e atualizar a   documentação |
| Possui                                   | 10064                   | 52,76%                     |

### Próximos Passos

1. Investigar registros cancelados.
2. Encontrar uma maneira de filtrar somente registros do Brasil (o campo area_publicacao não está preenchido em vários registros).
