# TechChallenge FIAP Fase 1
## Análise prévia sobre importação / exportação de vinhos, e influência nos resultados de seus fatores internos e externos
![image](https://cdn.dooca.store/624/posts/regioes-produtoras-de-vinho-no-mundo.jpg?v=1628169177)


Este projeto tem o objetivo de encontrar a influência de fatores internos e externos na importação e exportação de vinhos do Brasil em relação à outros países.


### Tratamento de dados
Os dados utilizados foram os disponibilizados no portal no COMEX STAT [Portal para acesso gratuito às estatísticas de comércio exterior do Brasil. Crie consultas detalhadas das exportações e importações brasileiras com as diversas variáveis da base de dados estatísticos.](http://comexstat.mdic.gov.br/en/geral/91544)). 

Ao analisar esses dados identifiquei a necessidade de alguns tratamentos dos dados:
* Retirar as linhas com marcação de UTI = 1
* Remarcar a coluna ICU (UTI em inglês), com a informação se o paciente visitante (PATIENT_VISIT_IDENTIFIER) chegou na UTI, independente da janela (campo WINDOW)
* Tratando os dados que não estão disponíveis com base na medição anterior ou posterior
  * Caso ainda sobre valores indisponíveis, as linhas serão excluídas
* Remover as colunas que tem o mesmo valor em todas as linhas da base
* Transformar o campo demográfico 'AGE PERCENTIL' que está em formato de texto
  * Aqui trabalhei com 2 formas diferentes e criei 2 bases para testes:
    * dados_LE utilizando o método  LabelEncoder() do scikit-learn. 
    * dados_OHE utilizando o método  OneHotEncoder() do scikit-learn. 

Um tratamento adicional ocorre quando queremos trabalhar com a janela seguinte (2-4 horas). Neste caso incluímos os dados das features contínuas da janela anterior, bem como a variação dessas mesmas features entre janelas.

### Testes para a criação do Pipeline
Nesta parte eu testei alguns tratamentos e percebi, por exemplo, que fazer a eliminação das variáveis com alta correlação entre si (mantendo apenas uma delas) logo no início, bem como retirar as que possuem baixa correlação com a variável alvo de UTI, já melhorava as previsões e reduzia o tempo de processamento.

Após isso, normalmente colocar mais uma seleção de dados, utilizando o modelo SelectFromModel, também melhorava as estimativas. Sendo que aqui um dos testes também foi encontrar o melhor parâmetro para o threshold desse modelo.

Outo teste interessande foi se era necessário reescalar os números. Logo de cara eu desconsiderei a função Normalizer por ser evidente que piorava os indicadores, desta forma o teste ficou entre não fazer nada (a base de certa forma já estava ajustada) ou utilizar o modelo StandardScaler.

Por fim o modelo a ser utilizado, testei tanto o Logistic Regression quanto o Random Forest, sendo que em ambos qual a parametrização que melhorava o resultado. No caso do Random Forest o tempo de processamento foi um entrave que limitou um pouco os testes dos parâmetros.

### Construção do Pipeline
Dado o resultado dos testes eu crio o pipeline de melhor performance, sendo que considerei o limite inferior do intervalo de confiança de 95% (media - 2 desvios padrões) do ROC AUC como referência.

Com o pipeline criado rodo para uma amostra de teste aleatória os resultados para ter uma referência do resultado. Fora isso, podemos salvar o modelo em um arquivo.
[Aqui você pode encontrar os arquivos criados]().

### Conclusões
Esses testes todos foram feitos tanto na janela de 0-2 quanto na janela de 2-4. Sendo que na janela de 0-2 horas o melhor modelo de Regressão Logística e do Random Forest ficaram com resultados iguais, impossibilitando a definição de um deles.

Como na janela de 2-4 o melhor foi o Random Forest, talvez o melhor seria seguir com ele para todas as janelas, mas valeria acompanhar ambos, dado os resultados próximos.

No arquivo da seleção dos modelos temos a lista dos campos selecionados, deixei lá mais por curiosidade de quais características podem influenciar na internação segundo cada modelo. Sendo que na janela de 2-4, os campos criados com a variação das features entre janelas tem uma participação importante em ambos, o que eu já imaginava, dado que a mudança nos exames é o que normalmente indica uma piora ou melhora no quadro do paciente.

Acredito que os modelos estão bons, mas talvez com mais alguns testes poderíamos encontrar modelos melhores. Com isso, as sugestões de próximos passos seriam:
* Testar outros modelos de classificação, dado que só utilizei 2 nos meus testes.
* Na parametrização do modelo acabei utilizando uma métrica diferente da utilizada na comparação dos pipelines, teria que fazer alguns ajustes nos códigos para que ela também fosse utilizada na parte de parametrização, deixando de ser o valor médio e sim o valor inferior do intervalo de confiança.
* Outro ponto que poderia ser melhorado é na hora de retirar as variáveis que tem alta correlação entre si, o certo era manter a que tem maior correlação com a variável y e atualmente está escolhendo conforme a ordem das colunas.

Acredito que esses são os principais pontos e até uma próxima!

### Principais Referências
[Bootcamp Alura](https://bootcamps.alura.com.br/)

[scikit-learn user guide](https://scikit-learn.org/stable/user_guide.html)

[Implementando um Modelo de Classificação no Scikit-Learn](https://tatianaesc.medium.com/implementando-um-modelo-de-classifica%C3%A7%C3%A3o-no-scikit-learn-6206d684b377) - Tatiana Escovedo
