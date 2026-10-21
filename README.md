## 2. Vias de acesso avaliadas

| Via | Funciona para 2024+? | Formato | Estabilidade | Esforço | Veredito |
|---|---|---|---|---|---|
| A. Web service `listarreceita` | **Não.** Só devolve dados até 2018 | JSON | Parado desde 2018 | Baixo | **Descartada** |
| B. Downloads / dados abertos | **Não.** O arquivo com os valores de receita está vazio | CSV | Parado | Baixo | **Descartada** |
| C. Exportação do FlexPortal | **Não.** O arquivo só existe dentro da sessão | CSV | Depende da sessão | — | **Descartada** |
| D. Raspagem da tela Vaadin | Para resolver | Para resolver | Para resolver | Para resolver | Para resolver |
| Pistas: SEFA / outras consultas do menu | Para resolver | Para resolver | Para resolver | Para resolver | Para resolver |

### 2.1 Via A — Web service

**O que foi feito.** Rodamos um script que chama o endereço `.../pte/ws/receitas/listarreceita` uma vez por vez, para os anos de 2016 a 2026, nos meses 1 e 12, com o órgão 45 e sem informar órgão (44 chamadas no total).

**Resultado.** Todas as chamadas responderam normalmente (HTTP 200).
- **2016, 2017 e 2018:** trazem dados. Exemplo: janeiro/2016 devolveu 2.306 registros com o órgão 45 e 4.550 sem órgão.
- **2019 a 2026:** todas as chamadas devolveram "Nenhum registro encontrado", com e sem órgão.

**Conclusão.** O serviço está no ar, mas parou de ser alimentado em 2018. Não é problema de código de órgão, porque a consulta sem órgão também vem vazia. A resposta traz 35 campos (por exemplo `valor_estimado`, `valor_arrecadado`, `valor_suplementado`, `valor_cancelado`), que servem de base para a §3.1.

**Limite do teste.** Testamos só os meses 1 e 12 e só o órgão 45.

### 2.2 Via B — Downloads / dados abertos

**O que foi feito.** Baixamos os arquivos de Receitas de 2023, 2024 e 2025 (7 arquivos por ano) e abrimos todos para ver quais têm dados.

**Resultado.**

| Ano | Arquivos com dados | Arquivos vazios |
|---|---|---|
| 2023 | Fonte de recursos, Grupo de fonte, Órgão, Unidade orçamentária | **Receita**, Natureza da receita, Subunidade orçamentária |
| 2024 | Fonte de recursos, Grupo de fonte, Órgão | **Receita**, Natureza da receita, Subunidade orçamentária, Unidade orçamentária |
| 2025 | Nenhum | Todos os 7 |

**Conclusão.** Os arquivos que têm dados são só listas de apoio (órgãos, fontes etc.). O arquivo com os valores de receita (`TB_RECEITA`) está vazio nos três anos. Em 2024 até a lista de unidades orçamentárias está vazia, então nem as universidades dá para localizar por aqui.

### 2.3 Via C — Exportação do FlexPortal

**O que foi feito.** Acessamos a tela de receitas do portal (`https://www.transparencia.pr.gov.br/FlexPortal/#!Receitas`) e conferimos como os dados são gerados.

**Resultado.** Os dados dependem da sessão: o arquivo só é gerado dentro da sessão aberta no navegador e não existe um link fixo para baixar depois.

**Conclusão.** Não dá para usar a exportação como via de acesso por conta própria. Descartada.

### 2.3 Conclusão parcial

Nem o web service nem os arquivos de Downloads entregam receita de 2024 em diante. Sobram as vias C e D (a tela do portal), que ainda precisam ser testadas.

**Via recomendada:** Para resolver · **Motivo em uma frase:** Para resolver





# Demo-CI: Aula Prática sobre Servidores de Integração Contínua

Este repositório descreve um roteiro prático para configuração e uso de um **Servidor de Integração Contínua**. O objetivo é proporcionar ao aluno um primeiro contato real com essa prática de desenvolvimento de software.

Se você ainda não sabe o que é **Integração Contínua** e também não entende o papel desempenhado por servidores de CI, recomendamos antes ler o [Capítulo 10](https://engsoftmoderna.info/cap10.html) do nosso livro texto ([Engenharia de Software Moderna](https://engsoftmoderna.info/cap10.html)).

Apesar de existirem diversos servidores de integração contínua, neste roteiro iremos usar um recurso nativo do GitHub, chamado **GitHub Actions**, para configurar um servidor de CI.

<p align="center">
    <img width="70%" src="https://user-images.githubusercontent.com/7620947/109080916-232f8200-76e0-11eb-8d02-9ca9f518cea2.png" />
</p>

O Github Actions permite executar programas externos assim que determinados eventos forem detectados em um repositório GitHub. Como nosso intuito é configurar um servidor CI, iremos usar o GitHub Actions para compilar todo o código (_build_) do projeto e rodar seus testes de unidade quando um Pull Request (PR) for aberto no repositório, conforme ilustrado a seguir.

<p align="center">
    <img width="70%" src="https://user-images.githubusercontent.com/7620947/110053018-50f77500-7d37-11eb-9ca1-3de609b93584.png" />
</p>

## Programa de Exemplo

Para ilustrar o uso do servidor de CI, vamos usar um programa Java muito simples, que já foi criado e está disponível neste mesmo repositório ([Calculadora.java](https://github.com/aserg-ufmg/demo-ci/blob/main/src/main/java/br/ufmg/dcc/Calculadora.java)):

```java
public class Calculadora {

  public int soma(int x, int y) {
    return x + y;
  }

  public int subtrai(int x, int y) {
    return x - y;
  }
}
```

Quando chegar um PR no repositório, o servidor de CI vai automaticamente realizar um _build_ desse programa e rodar o seguinte teste de unidade (também já disponível no repositório, veja em [CalculadoraTest.java](https://github.com/aserg-ufmg/demo-ci/blob/main/src/test/java/br/ufmg/dcc/CalculadoraTest.java)):

```java
public class CalculadoraTest {

  @Test
  public void testeSoma1() {
    Calculadora calc = new Calculadora();
    int resultadoEsperado = 5;
    int resultadoRetornado = calc.soma(2,3);
    assertEquals(resultadoEsperado, resultadoRetornado);
  }

  @Test
  public void testeSoma2() {
    Calculadora calc = new Calculadora();
    assertEquals(10, calc.soma(4,6));
  }
}
```

## Tarefa #1: Configurar o GitHub Actions

#### Passo 1

Antes de mais nada realize um fork deste repositório. Para isso, basta clicar no botão **Fork** no canto superior direito desta página.

Ou seja, você irá configurar um servidor de CI na sua própria cópia do repositório.

#### Passo 2

Clone o repositório para sua máquina local, usando o seguinte comando (onde `<USER>` deve ser substituído pelo seu usuário no GitHub):

```bash
git clone https://github.com/<USER>/demo-ci.git
```

Em seguida, copie o código a seguir para um arquivo com o seguinte nome: `.github/workflows/actions.yaml`. Isto é, crie diretórios `.github` e depois `workflows` e salve o código abaixo no arquivo `actions.yaml`.

```yaml
name: Github CI
on:
  # Configura servidor de CI para executar o pipeline de tarefas abaixo (jobs) quando 
  # um push ou pull request for realizado tendo como alvo a branch main
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  pipeline:
    runs-on: ubuntu-latest # Os comandos serão executados em um sistema operacional Linux

    steps:
      - name: Git Checkout
        uses: actions/checkout@v2 # Faz o checkout do código recebido

      - name: Set up JDK 1.8
        uses: actions/setup-java@v1 # Configura o Java 1.8
        with:
          java-version: 1.8

      - name: Build
        run: mvn package -Dmaven.test.skip=true # Compila o código fonte

      - name: Unit Test
        run: mvn test # Executada os testes de unidade
```

Esse arquivo ativa e configura o GitHub Actions para -- toda vez que ocorrer um evento `push` ou `pull_request` tendo como alvo a branch principal do repositório -- realizar três tarefas (jobs):

- realizar o checkout do código;
- realizar um build;
- rodar os testes de unidade.

#### Passo 3

Realize um `commit` e um `git push`, isto é:

```bash
git add --all
git commit -m "Configurando GitHub Actions"
git push origin main
```

#### Passo 4

Quando o `push` chegar no repositório principal, o GitHub Actions iniciará automaticamente o fluxo de tarefas configurado no arquivo `actions.yaml` (isto é, build + testes).

Você pode acompanhar o status dessa execução clicando na aba Actions do seu repositório.

<p align="center">
    <img width="80%" src="https://user-images.githubusercontent.com/7620947/110059807-b8b3bd00-7d43-11eb-9e57-e6ba1fa3457a.png" />
</p>

## Tarefa #2: Criando um PR com bug

Para finalizar, vamos introduzir um pequeno bug no programa de exemplo e enviar um PR, para mostrar que ele será "barrado" pelo processo de integração (isto, o nosso teste vai "detectar" o bug e falhar).

#### Passo 1

Introduza um pequeno bug na função `soma` do arquivo [src/main/java/br/ufmg/dcc/Calculadora.java](https://github.com/rodrigo-brito/roteiro-github-actions/blob/main/src/main/java/br/ufmg/dcc/Calculadora.java). Por exemplo, basta alterar a linha 6, alterando o retorno da função para `x + y + 1`, como apresentado abaixo.

```diff
--- a/src/main/java/br/ufmg/dcc/Calculadora.java
+++ b/src/main/java/br/ufmg/dcc/Calculadora.java
@@ -3,7 +3,7 @@ package br.ufmg.dcc;
 public class Calculadora {

   public int soma(int x, int y) {
-    return x + y;
+    return x + y + 1;
   }

   public int subtrai(int x, int y) {
```

#### Passo 2

Após modificar o código, você deve criar um novo branch, realizar um `commit` e `push`:

```bash
git checkout -b bug
git add --all
git commit -m "Incluindo alterações na função soma"
git push origin bug
```

#### Passo 3

Em seguida, crie um Pull Request (PR) com sua modificação. Para isso, basta acessar a seguinte URL em seu navegador: `https://github.com/<USER>/demo-ci/compare/main...bug`, onde `<USER>` deve ser substituido pelo seu usuário no GitHub. Nessa janela, você pode conferir as modificações feitas e incluir uma pequena descrição no PR.

<p align="center">
    <img width="70%" src="https://user-images.githubusercontent.com/7620947/111704705-5b793a80-881e-11eb-8422-22d51bde6b19.png" />
</p>

Após finalizar a criação do Pull Request, será iniciada nossa pipeline, ou seja, o próprio GitHub vai fazer o build do sistema e rodar seus testes (como na tarefa #1). Porém, dessa vez os testes não vão passar, como mostrado abaixo:

<p align="center">
    <img width="70%" src="https://user-images.githubusercontent.com/7620947/111704932-a85d1100-881e-11eb-8d3b-31f34bafa986.png" />
</p>

**RESUMINDO**: O Servidor de CI conseguiu alertar, de forma automática, tanto o autor do PR como o integrador de que existe um problema no código submetido, o que impede que ele seja integrado no branch principal do repositório.

## Créditos

Este roteiro foi elaborado por **Rodrigo Brito**, aluno de mestrado do DCC/UFMG, como parte das suas atividades na disciplina Estágio em Docência, cursada em 2020/2, sob orientação do **Prof. Marco Tulio Valente**.
