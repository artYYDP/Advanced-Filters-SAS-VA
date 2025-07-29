# 🔍 Filtrando termos em diferentes colunas

## 🧠 Objetivo

Permitir que o usuário insira livremente um termo (por exemplo, "Maria" ou "12345") e que esse termo seja pesquisado em **mais de uma coluna** da mesma tabela — por exemplo: `Nome` **ou** `CPF`.

## 🧩 Exemplo prático

Imagine uma tabela com os nomes dos estados brasileiros (UFs) e municípios, onde queremos que o mesmo filtro de texto procure tanto pelo nome do estado quanto pelo nome do município.

Para este exemplo prático, usaremos uma tabela de municípios do IBGE. Esta tabela contém os seguintes dados:

| Coluna | Descrição | Tipo |
| - | - | - |
| CD_UF | Código do estado | Numérico |
| DS_UF | Descrição do estado | Caractere |
| CD_MUN | Código do município | Numérico |
| DS_MUN | Descrição do município | Caractere |

Clique [Baixar MUNICIPIOS.csv](/files/MUNICIPIOS.csv) para obter o arquivo CSV e faça o upload do arquivo para o ambiente SAS Viya. Em seguida, use o código abaixo para carregar a tabela na memória no `PUBLIC` CASLIB:

```sas
/* Defina o caminho do arquivo CSV */
%let filepath = [file_location]/MUNICIPIOS.csv;

/* Importar o arquivo CSV */
proc import datafile="&filepath." out=PUBLIC.MUNICIPIOS dbms=csv replace;
  delimiter=';';
  getnames=yes;
  guessingrows=32767;
run;

/* Promover a tabela para o CASLIB Público */
proc casutil;
  promote incaslib="PUBLIC" casdata="MUNICIPIOS" outcaslib="Public";
quit;
```

Agora com nossa tabela na memória, podemos ir ao SAS Visual Analytics (VA) e criar nossa busca.

Na tela inicial do SAS VA, vamos criar um novo relatório e vincular a fonte de dados que carregamos na memória.

![01](https://www.dropbox.com/scl/fi/ymvrzlje0gqglusyuw4qv/01.png?rlkey=dy0ht1jmfs0y6oc90c9uizx23&st=dvvns0xy&raw=1)

![02](https://www.dropbox.com/scl/fi/twsjb0s9a9elw68rlte1w/02.png?rlkey=zurwhhsch4ckiollfvcdfbynp&st=4hcma1dx&raw=1)

Clique no ícone de Objetos (o segundo ícone à esquerda, de cima para baixo) e procure por `texto` nos filtros. Selecione e arraste a **Entrada de Texto** para a área do SAS:

![03](https://www.dropbox.com/scl/fi/gx0eqpxnlvwjdfcstqx8f/03.png?rlkey=wix9rl6yi6orb62nvj3abhs76&st=e5lfxpod&raw=1)

![04](https://www.dropbox.com/scl/fi/ukxicsrrz5f2v2o8yj2us/04.png?rlkey=nkd5swa41x3in7wodmenihsmd&st=t8f949ru&raw=1)

Em seguida, volte para a mesma tela e selecione a **Tabela de Lista**, arrastando-a para o painel:

![05](https://www.dropbox.com/scl/fi/mwtkmjnlqynpyo52doxys/05.png?rlkey=ckpkw1sm8rpfkn4y48l0r1eqk&st=oiwoypwv&raw=1)

Clique em **Atribuir dados** na tabela e configure conforme mostrado abaixo:

![06](https://www.dropbox.com/scl/fi/24drtzbwz4yan8g27nxle/06.png?rlkey=1236kxwgc2i95xxfr3v87nsdx&st=mnwlevaa&raw=1)

![07](https://www.dropbox.com/scl/fi/ksz3obt3cn98k2726gk00/07.png?rlkey=xdyi0ipehtzbnpyko4sphips2&st=xxwggtzv&raw=1)

![08](https://www.dropbox.com/scl/fi/2eww2905ism5997c42ad3/08.png?rlkey=i6pjes26yko05kche6gf5sm3f&st=xuf7padu&raw=1)

Agora, vamos adicionar um parâmetro para permitir a busca através dele. Para isso, vá em **Dados > Novo item de dados > Parâmetro**:

![09](https://www.dropbox.com/scl/fi/65abdhe4ziwjvmlflr0xk/09.png?rlkey=fx8q4s2x0lno2k1f40gp2yh75&st=8w9qq0wv&raw=1)

Nomeie o parâmetro e defina-o como `Caractere`, deixando o valor atual vazio:

![10](https://www.dropbox.com/scl/fi/ximazapqhn0lxo8iik7yn/10.png?rlkey=d9lsks8nv957lccw1kuvq282f&st=k5403bbc&raw=1)

![11](https://www.dropbox.com/scl/fi/v7y0yd7sqm6zuc1ipjq4y/11.png?rlkey=pzjmcg4g0yn5ig1sgfg7qbjdw&st=v9f3typi&raw=1)

Em seguida, no campo **Atribuir dados** da caixa de texto, atribua o parâmetro criado:

![12](https://www.dropbox.com/scl/fi/jwgkjc025dim0h1h7v4wy/12.png?rlkey=thsu8bm4rhxtdrhdxtqy8ch3e&st=lo70nuga&raw=1)

![13](https://www.dropbox.com/scl/fi/4y26iko9eak5kehrsszjt/13.png?rlkey=gdvesuoregepyfolmpt8mh0uv&st=ecr3p6d1&raw=1)

Em seguida, selecione a tabela, clique no ícone de filtro no canto superior direito da tela e escolha **Novo filtro**, selecionando a opção **Filtro avançado**:

![15](https://www.dropbox.com/scl/fi/9hnb7o9pkoxpz59oa9fki/15.png?rlkey=5y1ouijm1cvjmng080ychwos0&st=onnub4zv&raw=1)

Nesta parte da **Expressão de filtro**, é aqui que a mágica acontece para que tudo funcione corretamente. Vou explicar cada detalhe separadamente para garantir que você compreenda totalmente o que estamos fazendo aqui.

A expressão de filtro requer uma condição, e é exatamente isso que vamos aplicar. A condição que usaremos é a seguinte:

```plaintext
OR [
  FiltroGeral Missing
  OR [
    LowerCase(DS_UF) Contains LowerCase(FiltroGeral)
    LowerCase(DS_MUN) Contains LowerCase(FiltroGeral)
  ]
]
```

- FiltroGeral: é o parâmetro que você criou para ser usado na busca;
- OR: representa a condição "OU", ou seja, a expressão será verdadeira se qualquer uma das condições for atendida;
- Missing: faz com que a condição retorne vazia quando nenhum valor for fornecido, permitindo que todos os registros apareçam na tabela;
- LowerCase: converte o texto para minúsculas, garantindo que a busca seja case-insensitive. O mesmo efeito pode ser alcançado com UpperCase;
- Contains: verifica se o campo contém o texto especificado, útil para buscas parciais.

Aplique a condição exatamente como mostrado na imagem abaixo:

![16](https://www.dropbox.com/scl/fi/htpvpriszu7p6wgq46io4/16.png?rlkey=o7pd3cvezjxc7a4jhbi85wre6&st=g6qxmu9a&raw=1)

Em seguida, clique em OK e teste sua nova barra de busca:

![17](https://www.dropbox.com/scl/fi/qszpqsywasbp2erymhlzn/17.png?rlkey=0xgz8wpb17jeeosafvwtx28en&st=kl10t991&raw=1)

Se desejar, você pode expandir a busca para mais colunas. Para isso, basta adicionar mais condições OR e seguir a mesma estrutura da condição anterior.

## Agradecimentos

- [Geiziane Oliveira](https://www.linkedin.com/in/geiziane-oliveira-0a5882110/) - Mentora SAS, cuja orientação foi essencial para a construção deste parâmetro.

## Referências

- [Entrada de Texto vinculada a múltiplas categorias no SAS VA](https://communities.sas.com/t5/SAS-Visual-Analytics/Text-Input-linked-to-multiple-categories-in-SAS-VA/m-p/784471#M15682)