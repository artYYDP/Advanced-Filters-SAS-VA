# 🔍 Filtrando termos em colunas diferentes

## 🧠 Objetivo

Permitir que o usuário digite um termo livremente (ex: "Maria" ou "12345") e que esse termo seja buscado em **mais de uma coluna** da mesma tabela — por exemplo: `Nome` **ou** `CPF`.

## 🧩 Exemplo prático

Imagine uma tabela com os nomes das UF's (Unidades Federativas) e municípios, onde queremos que o mesmo filtro de texto procure, tanto pelo nome da UF, quanto pelo nome do município no mesmo filtro.

Para esse exemplo prático, usaremos uma tabela de municípios do IBGE. Essa tabela contém os seguintes dados:

| Coluna | Descrição | Tipo |
| - | - | - |
| CD_UF | Código da UF | Numeric |
| DS_UF | Descrição da UF | Character |
| CD_MUN | Código da UF | Numeric |
| DS_MUN | Descrição da UF | Character |

Clique [aqui](/files/MUNICIPIOS.csv) e baixe o CSV e suba o arquivo no seu ambiente SAS Viya. Depois, utilize o código abaixo para colocar a tabela em memória na CASLIB `PUBLIC`:

```sas
/* Definir o caminho do arquivo CSV */
%let filepath = [local_do_arquivo]/MUNICIPIOS.csv;

/* Importar o arquivo CSV */
proc import datafile="&filepath." out=PUBPLIC.MUNICIPIOS dbms=csv replace;
	delimiter=';';
	getnames=yes;
  guessingrows=32767;
run;

/* Promover a tabela para a CASLIB Public */
proc casutil;
	promote incaslib="PUBLIC" casdata="MUNICIPIOS" outcaslib="Public";
quit;
```

Agora com a nossa tabela em memória, podemos ir para o SAS Visual Analytics (VA) e criarmos a nossa pesquisa.

Na tela inicial do SAS VA, vamos criar um novo relatório e atrelar a fonte de dados que colocamos em memória.

![01](/images/FilterColunms/01.png)

![02](/images/FilterColunms/02.png)

Clique no ícone de Objetos (segundo ícone do lado esquerdo, de cima para baixo) e procure por `texto` nos filtros. Selecione e arraste para a área do SAS a **Entrada de texto**:

![03](/images/FilterColunms/03.png)

![04](/images/FilterColunms/04.png)

Em seguida, volte à mesma tela e selecione a **Tabela de listas**, arrastando-a para o painel:

![05](/images/FilterColunms/05.png)

Clique em **Atribuir dados** na tabela e configure conforme mostrado abaixo:

![06](/images/FilterColunms/06.png)

![07](/images/FilterColunms/07.png)

![08](/images/FilterColunms/08.png)

Agora, vamos adicionar um parâmetro para permitir a busca através dele. Para isso, vá em **Dados > Novo item de dados > Parâmetro**:

![09](/images/FilterColunms/09.png)

Nomeie o parâmetro e defina como `Caractere`, deixando o valor atual vazio:

![10](/images/FilterColunms/10.png)

![11](/images/FilterColunms/11.png)

Em seguida, no campo **Atribuir dados** da caixa de texto, atribua o parâmetro criado:

![12](/images/FilterColunms/12.png)

![13](/images/FilterColunms/13.png)

Na sequência, selecione a tabela, clique no ícone de filtro no canto direito da tela e escolha **Novo filtro**, selecionando a opção **Filtro avançado**:

![15](/images/FilterColunms/15.png)

Nessa parte da **Expressão do filtro**, é onde a mágica acontece para que tudo funcione corretamente. Vou explicar cada detalhe separadamente para garantir que você entenda completamente o que estamos fazendo aqui.

A expressão de filtro exige uma condição, e é exatamente isso que vamos aplicar. A condição que vamos usar é a seguinte:

```plaintext
OR [
  FiltroGeral Missing
  OR [
    LowerCase(DS_UF) Contains LowerCase(FiltroGeral)
    LowerCase(DS_MUN) Contains LowerCase(FiltroGeral)
  ]
]
```

- FiltroGeral: é o parâmetro que você criou para ser utilizado na pesquisa;
- OR: representa a condição "OU", ou seja, a expressão será verdadeira se qualquer uma das condições for atendida;
- Missing: faz com que a condição retorne vazio quando nenhum valor for informado, permitindo que todos os registros apareçam na tabela;
- LowerCase: converte o texto para minúsculas, garantindo que a pesquisa não faça distinção entre letras maiúsculas e minúsculas. O mesmo efeito pode ser alcançado com UpperCase;
- Contains: verifica se o campo contém o texto especificado, sendo útil para buscas parciais.

Aplique a condição exatamente como mostrado na imagem abaixo:

![16](/images/FilterColunms/16.png)

Depois, clique em OK e teste a sua nova barra de pesquisa:

![17](/images/FilterColunms/17.png)

Se desejar, você pode expandir a busca para mais colunas. Para isso, basta adicionar mais condições OR e seguir a mesma estrutura da condição anterior.

## Agradecimentos

- [Geiziane Oliveira](https://www.linkedin.com/in/geiziane-oliveira-0a5882110/) - Mentora SAS, cuja orientação foi fundamental para a construção desse parâmetro.

## Referências

- [Text Input linked to multiple categories in SAS VA](https://communities.sas.com/t5/SAS-Visual-Analytics/Text-Input-linked-to-multiple-categories-in-SAS-VA/m-p/784471#M15682)