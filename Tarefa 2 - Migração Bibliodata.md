## Migração Bibliodata

### 1. Investigação

O objetivo desta tarefa é a de investigar os seguintes pontos:
1. Não veio o campo 100 - Nome pessoal
2. Não veio o campo 110 - Autoria de entidade
3. Não veio o campo 111 - Autoria de evento
4. Registros duplicados.
5. Registros sem título.

Abaixo, encontra-se a tabela resumo do resultado da investigação.

|           Ponto a   ser investigado          |   Método utilizado   |          Resultado         |                                      Ação                                      |
|:--------------------------------------------:|:--------------------:|:--------------------------:|:------------------------------------------------------------------------------:|
| Não veio o campo 100 - Nome pessoal          | Documentação/Dbeaver | Não consta na documentação | Desenvolver script, alterar a tabela de registros e atualizar a   documentação |
| Não veio o campo 110 - Autoria de   entidade | Documentação/Dbeaver | Não consta na documentação | Desenvolver script, alterar a tabela de registros e atualizar a   documentação |
| Não veio o campo 111 - Autoria de evento     | Documentação/Dbeaver | Não consta na documentação | Desenvolver script, alterar a tabela de registros e atualizar a   documentação |
| Registros duplicados                         | -                    | -                          | -                                                                              |
| Registros sem título                         | -                    | -                          | -                                                                              |

### 2. Novos Scripts

Após a reunião do sprint, conversei com o Lucas Matos e consegui um script para alteração da tabela de maneira a adicionar novas três colunas. Outro tipo de script seria para adicionar o valor dos respectivos campos (100, 110 e 111) na tabela de registros bibliográficos.

Dito isto, para cada campo, criei um script que acrescenta a coluna necessária, realiza o enriquecimento da tabela e trata esse valor adicionado. Abaixo, encontra-se um dos três scripts desenvolvidos:
```sql
ALTER TABLE registro_bibliografico 
ADD COLUMN autoria_evento VARCHAR(255);

CREATE TEMPORARY TABLE temporaria (
  codigo INTEGER,
  autoria_evento TEXT
);

COPY temporaria (codigo, autoria_evento)
FROM 'K:\Download\Bibliodata\scripts_bibliodata\CSVs\CAMPO111.csv'
DELIMITER ';' CSV HEADER;

UPDATE registro_bibliografico
	SET codigo = COALESCE(t.codigo, registro_bibliografico.codigo),
    	autoria_evento = COALESCE(t.autoria_evento, registro_bibliografico.autoria_evento)
	FROM temporaria t
WHERE registro_bibliografico.codigo = t.codigo;

-- regexp_split_to_array para dividir a coluna "autoria_evento" em um array de strings usando o caractere "" como delimitador.
-- verifica se o comprimento do array resultante é maior ou igual a 2 (ou seja, se existem pelo menos duas ocorrências do caractere "").
-- se for maior ou igual a 2, é selecionado na lista o item correspondente a variável "título"
UPDATE registro_bibliografico
SET autoria_evento = (
    CASE
        WHEN array_length(regexp_split_to_array(autoria_evento, ''), 1) >= 2 THEN
            substring((regexp_split_to_array(autoria_evento, ''))[1] || '' || (regexp_split_to_array(autoria_evento, ''))[2] FROM 3)
        ELSE
            autoria_evento
    END
);

DROP TABLE temporaria;
```




O resultado das operações realizadas foram coerentes com o resultado desejado, a exemplo:

![image](https://github.com/user-attachments/assets/b46c183a-b086-40d9-afd7-900cd237dd7b)

Obs: o [repositório]() foi atualizado com os novos scripts.

### Reunião com Lucas Matos - 08/11/2024

Em conversa com o Lucas Matos, marcamos uma conversa na sexta-feira onde ele me apresentou toda a estrutura do código, como ela funcionava e realizou ajustes para que eu possa testar na minha máquina. De começo, precisei instalar algumas libs e configurar as credenciais da conexão. Após isso, executei o programa:

![image](https://github.com/user-attachments/assets/210af0d5-3b23-4233-93ff-1703bfca1170)

Apesar de conseguir gerar, ainda preciso alterar o script para retornar os campos 100, 110 e 111:

![image](https://github.com/user-attachments/assets/33aa6cf2-1803-4607-a9ab-58c1fed1b398)

Depois de depurar o código, consegui ajustar de maneira a trazer o valor do campo desejado:

![image](https://github.com/user-attachments/assets/1ecb7ea6-9c91-40f8-887f-6ffc07889498)

Como o registro está no banco de dados local:

![image](https://github.com/user-attachments/assets/92c8eebe-e8a5-40f1-92d0-c37d4912b4b4)

## Migração Bibliodata

O meu objetivo nesta sprint era realizar a migração do banco legado do Biliodata e disponibilizar para o Alyson.

Esta migração se dá por meio de duas etapas:
1. Exportaçao dos campos MARC21 em arquivos CSVs.
2. Executar uma série de scripts SQL.

A migração em si deveria ser rápida, algo que falei na reunião passada, porém, ao começar a primeira etapa, percebi que o buraco é bem mais embaixo.

De quinta-feira até sábado, consegui exportar cerca de 300 MB de arquivo (em 2023 foram exportados estes arquivos, com um tamanho de 1,14 GB), o que representa menos de 30% do que deveria ser gerado.

Problemas enfrentados:
1. Queda da conexão com a VNP (acontecia por volta de cada 4h de duração de conexão): 
![image](https://github.com/user-attachments/assets/8897f4a1-4fd1-495d-b210-d30abbb77bed)

2. Conexão lenta com o banco legado:
![image](https://github.com/user-attachments/assets/c23169d7-57ea-4dae-a891-3e8166c89d11)

3. Lentidão para realizar selects:
![image](https://github.com/user-attachments/assets/8d11dfc4-3ba3-46f4-9bde-00019c0ed56d)
