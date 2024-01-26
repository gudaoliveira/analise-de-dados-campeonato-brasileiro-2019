
## ANÁLISE ESTATÍSTICA - A INFLUÊNCIA DA LONGEVIDADE DE UM TÉCNICO EM UM TIME NO BRASILEIRÃO DE 2019
### "Lógica de programação 2 com Python" - ADA Tech & IFood 🎲

## Integrantes: 👥

-   #### Alan Gonçalves
-   #### Élen Silva Almeida
-   #### Gabriel Matina
-   #### Gustavo Dell Anhol Oliveira
-   #### Patrick Kwan

## Descrição do problema🤔

Uma posição _volátil_, no futebol (tanto no Brasil quanto fora), é a posição de técnico. Uma prática comum é substituir o treinador após alguns resultados negativos. Naturalmente, um time que não troca o técnico não é garantia de sucesso no campeonato. Assim, surge a dúvida: 

<hr>
<div style="text-align:center; font-size:20px">
Será que, <b>em geral</b>, times com técnicos mais longevos ficam em uma posição melhor na tabela, no fim do campeonato?    
</div>
<hr>

## Manipulando os Dados🎲

Para _atacar_ esse problema, será disponibilizado um arquivo json com todos os dados do Campeonato Brasileiro de 2019. Para ler um arquivo json, basta importar o módulo 'pandas' com o comando:  ```import pandas as pd```. Para ler um arquivo json, utilize o método:  
```dados = pd.read_json('nome_do_arquivo.json') ```

Para acessar os valores, basta olhar os nomes presentes nas linhas e colunas. Por exemplo: 

```
dados[2][1], sendo que:
Primeiro índice -> Número da rodada
Segundo índice  -> Número da partida

```
Isso nos dá acesso a um dicionário com todas as informações da partida em questão, que nos permite acessar os valores utilizando as suas chaves.

```
#Nós importamos os dados utilizando a biblioteca Pandas para visualizarmos o arquivo json com a tabela
import pandas as pd

dados = pd.read_json('brasileirao-2019.json')
dados
```

## Tratamento dos dados📝:

Para começar a nossa análise primeiro começamos a tratar os dados que nos foram fornecidos. Para obter a resposta, precisamos dos seguintes dados:

#### Em relação as partidas
- Uma lista com o nome de todos os Treinadores
- Uma lista com o nome de todos os times

#### Em relação aos times
- As pontuações e demais estatísticas da partida
- A sua classificação baseada na pontuação
- Todos os técnicos que passaram pelo time, e a quantidade de jogos em que atuaram com os mesmos
- Qual foi o técnico que mais atuou em cada time e a quantidade de jogos

#### Encontrando os técnicos

Para consultarmos o técnico de um determinado time, podemos acessar o dataframe da seguinte forma:
```
dados[número da rodada][número do jogo]['coach']
(O índice 'coach' é uma chave do dicionário de informações presente na célula em questão)
```
Com isso, podemos criar uma lista com todos os técnicos

```
#Criamos uma lista para armazenar o nome de todos os treinadores do campeonato
lista_tecnicos = []

#Este for irá percorrer todos os dados e ir armazenando os treinadores com base nas partidas jogadas
for rodada in dados: #1 ao 38
    for jogo in range(len(dados[rodada])): #1 ao 10
        lista_tecnicos.append(dados[rodada][jogo]['coach']['home'])
        lista_tecnicos.append(dados[rodada][jogo]['coach']['away'])

lista_tecnicos = list(set(lista_tecnicos)) #Elimina as duplicas utilizando o set
lista_tecnicos
```
#### Encontrando os times

Com isso, da mesma forma, também podemos criar uma lista com todos os times. 

Também se atentamos em selecionar todos os nomes dentro das chaves 'home' e 'away'(que representam times que jogaram em casa e fora respectivamente), para ter certeza de que nenhum nome ficaria de fora

```
# Lista com o nome de todos os times
lista_times = []

for rodada in dados: #1 ao 38
    for jogo in range(len(dados[rodada])): #1 ao 10
        lista_times.append(dados[rodada][jogo]['clubs']['home'])
        lista_times.append(dados[rodada][jogo]['clubs']['away'])

lista_times = list(set(lista_times)) #Elimina as duplicas utilizando o set
lista_times
```

E com essa informação, conseguimos criar uma estrutura de dados para relacionar o nome do time, os técnicos e todos os jogos que os mesmos atuaram entre si

```
# Para armazenarmos a quantidade de jogos que um treinador teve para os respectivos clubes que ele treinou, criamos uma nova variável
partidas = {}

# Este for cria um dicionário com o nome de cada um dos times, os seus respectivos treinadores e 
# quantas partidas o treinador atuou pelo time (inicialmente 0) 
for coach in lista_tecnicos:
    for time in lista_times:
        partidas[time] = {coach : 0 for coach in lista_tecnicos}
        
for coach in lista_tecnicos:
    for time in lista_times:
        partidas[time][coach] = 0

# Aqui preenchemos a quantidade de vezes que o técnico treinou o time
for rodada in dados: #1 ao 38
    for jogo in range(len(dados[rodada])): #0 ao 9
        time_casa = dados[rodada][jogo]['clubs']['home']
        coach_casa = dados[rodada][jogo]['coach']['home']
        
        time_fora = dados[rodada][jogo]['clubs']['away']
        coach_fora = dados[rodada][jogo]['coach']['away']
        
        partidas[time_casa][coach_casa] += 1
        partidas[time_fora][coach_fora] += 1
        
partidas
```

Com o dicionário ```partidas``` preenchido, filtramos os técnicos, deixando somente os mais longevos de cada time

```
mais_longevo = {}

for time in lista_times:
    partida_mais_longeva = 0
    mais_longevo[time] = {}  
    for coach in lista_tecnicos:
        if partidas[time][coach] > partida_mais_longeva:
            partida_mais_longeva = partidas[time][coach]
            mais_longevo[time] = {coach: partida_mais_longeva}
mais_longevo
```

Também precisamos das pontuações do time para definirmos a classificação, com isso criamos uma outra estrutura de dados chamada ```tabela_times``` que apenas extrai os respectivos dados do dataframe e os armazena

```
tabela_times = {}

for i in lista_times:
    tabela_times[i] = {'Jogos': 0,  # Qtde de jogos
                       'PTS': 0,    # Qtde de pontos totais
                       'VIT': 0,    # Qtde de vitórias
                       'EMP': 0,    # Qtde de empates
                       'DER': 0,    # Qtde de derrotas
                       'GM': 0,     # Qtde de gols marcados
                       'GC': 0,     # Qtde de gols contra (sofridos)
                       'SG': 0}     # Saldo de Gols

for rodada in dados: #1 ao 38
    for jogo in range(len(dados[rodada])): #0 ao 9
        
        time_casa = dados[rodada][jogo]['clubs']['home']
        time_fora = dados[rodada][jogo]['clubs']['away']
        
        golcasa = int(dados[rodada][jogo]["goals"]["home"])
        golfora = int(dados[rodada][jogo]["goals"]["away"])

        if golcasa > golfora:
            #REFERENTE AO TIME DA CASA:
            tabela_times[time_casa]["PTS"] += 3
            tabela_times[time_casa]["VIT"] += 1
            tabela_times[time_casa]["Jogos"] += 1
            tabela_times[time_casa]["GM"] += golcasa 
            tabela_times[time_casa]["GC"] += golfora
            tabela_times[time_casa]["SG"] += golcasa - golfora
            
            #REFERENTE AO TIME FORA
            tabela_times[time_fora]["DER"] += 1
            tabela_times[time_fora]["Jogos"] += 1
            tabela_times[time_fora]["GM"] += golfora 
            tabela_times[time_fora]["GC"] += golcasa
            tabela_times[time_fora]["SG"] += golfora - golcasa
            
        elif golcasa < golfora:
            #REFERENTE AO TIME FORA:
            tabela_times[time_fora]["PTS"] += 3
            tabela_times[time_fora]["VIT"] += 1
            tabela_times[time_fora]["Jogos"] += 1
            tabela_times[time_fora]["GM"] += golfora
            tabela_times[time_fora]["GC"] += golcasa
            tabela_times[time_fora]["SG"] += golfora - golcasa
            
            #REFERENTE AO TIME CASA
            tabela_times[time_casa]["DER"] += 1
            tabela_times[time_casa]["Jogos"] += 1
            tabela_times[time_casa]["GM"] += golcasa 
            tabela_times[time_casa]["GC"] += golfora
            tabela_times[time_casa]["SG"] += golcasa - golfora
            
        else:
            #REFERENTE AO TIME DA CASA:
            tabela_times[time_casa]["PTS"] += 1
            tabela_times[time_casa]['EMP'] += 1
            tabela_times[time_casa]["Jogos"] += 1
            tabela_times[time_casa]["GM"] += golcasa 
            tabela_times[time_casa]["GC"] += golfora
            tabela_times[time_casa]["SG"] += golcasa - golfora
            
            #REFERENTE AO TIME FORA
            tabela_times[time_fora]["PTS"] += 1
            tabela_times[time_fora]['EMP'] += 1
            tabela_times[time_fora]["Jogos"] += 1
            tabela_times[time_fora]["GM"] += golfora 
            tabela_times[time_fora]["GC"] += golcasa
            tabela_times[time_fora]["SG"] += golfora - golcasa

tabela_times
```
Com a pontuação definida, criamos a lista ```classificacao``` para definir a classificação dos times no campeonato utilizando o algoritimo de ordenação ```Bubble Sort``` para ordenar a lista do menor para o maior baseado em suas pontuações 

```
classificacao = [['Time','Pontos','Vitórias','Empates','Derrotas','Gols Marcados','Gols Sofridos','Saldo de Gols']]

for time in tabela_times:
    classificacao.append([time, tabela_times[time]['PTS'], tabela_times[time]['VIT'], tabela_times[time]['EMP'], tabela_times[time]['DER'],tabela_times[time]['GM'],tabela_times[time]['GC'],tabela_times[time]['SG']])
    
# Dentro da tupla
# 0 - Time              (nome)
# 1 - Pontos            (PTS)
# 2 - Vitórias          (VIT)
# 3 - Empates           (EMP)
# 4 - Derrotas          (DER)
# 5 - Gols marcados     (GM)
# 6 - Gols Sofridos     (GC)
# 7 - Saldo de gols     (SG)

# 
for i in range(1, len(classificacao)):
    for j in range(1, len(classificacao) - i):
        if classificacao[j][1] < classificacao[j + 1][1]:
            classificacao[j], classificacao[j + 1] = classificacao[j + 1], classificacao[j]

classificacao = list(enumerate(classificacao))
classificacao
```

No final observamos que houveram algumas situações de empates:
```
Empates em pontuação:
Santos e Palmeiras        74pts 
Vasco da Gama e Bahia     49pts
Chapecoense e CSA         32pts

```
Neste caso o Vasco da Gama ficou antes do Bahia e o Chapecoense ficou antes do CSA. Se analisarmos a classificação do Brasileirão de 2019 na internet, notamos que essa ordem é inversa, por conta do critério de desempate utilizado no campeonato, que resumido se baseia em:

- Maior quantidade de vitórias
- Maior saldo de gols     
- Maior quantidade de gols marcados

Então, de volta ao ```Bubble Sort```, aplicamos os critérios de desmpate e reordenamos a lista

```
# Critérios de desempate (NESTA ORDEM) 
#   Maior quantidade de vitórias        indice[2]
#   Maior saldo de gols                 indice[7]
#   Maior quantidade de gols marcados   indice[5]

for i in range(1, len(classificacao)):
    for j in range(1, len(classificacao) - i):
        if (classificacao[j][1] == classificacao[j + 1][1]):
            
            # Verifica se a Quantidade de vitórias é igual ou menor
            if (classificacao[j][2] < classificacao[j + 1][2]):
                classificacao[j], classificacao[j + 1] = classificacao[j + 1], classificacao[j]
            
            #Se a vitoria for igual, muda o criterio de desempate:
            elif (classificacao[j][2] == classificacao[j + 1][2]):
                # Se a quantidade do 'Saldo de gols' do item j for menor do que o próximo item
                if classificacao[j][7] < classificacao[j + 1][7]:
                    classificacao[j], classificacao[j + 1] = classificacao[j + 1], classificacao[j]

                # Se empatar no saldo de gols, verificar gols marcados
                elif classificacao[j][7] == classificacao[j + 1][7]:
                    # Se a quantidade de gols marcados do item j for menor do que o próximo item
                    if classificacao[j][5] < classificacao[j + 1][5]:
                        classificacao[j], classificacao[j + 1] = classificacao[j + 1], classificacao[j]
```

Com isso temos a classificação final do campeonato com as respectivas pontuações

```
dados_impressao = [(item[0], item[1]) for item in classificacao]
for row in dados_impressao:
    print(f"{row[0]:<3} | {row[1][0]:<15} | {row[1][1]:<10} | {row[1][2]:<10} | {row[1][3]:<10} | {row[1][4]:<10} | {row[1][5]:<15} | {row[1][6]:<15} | {row[1][7]}")
```

E utilizando as informações tratadas anteriormente, também conseguimos visualizar os times em ordem de classificação por pontos e seus respectivos técnicos mais longevos com a quantidade de partidas que o mesmo atuou no time

```
class_tecnico_longevo = []
tecnicos = list(mais_longevo.values())
for i in range(1, len(classificacao)):
    for chave, valor in mais_longevo.items():
        for treinador in valor.keys():
            if classificacao[i][1][0] == chave:
                print(f"Colocação:{classificacao[i][0]:2d} | {classificacao[i][1][0]:<15} | Técnico mais longevo:   {treinador:<28} | Qtde de partidas: {mais_longevo[chave][treinador]}")
```

Com as classificações e a quantidade de partidas que o técnico mais longevo atuo no time que fiocu na determinada classificação, podemos relacionar esses dados em um grafico de linhas para observar se há alguma relação entre a longevidade dos técnicos nos times e sua classificação

```
# classificacao[x][1][0]      Acessa os nomes dos times
# classificacao[1][0]         Acessa a classificação dos times

import matplotlib.pyplot as plt

dados_plotagem = []
dados_plotagem_x = []
dados_plotagem_y = []

for i in range(1, len(classificacao)):
    nome_time = classificacao[i][1][0]
    tec_mais_longevo = list(mais_longevo[nome_time].items())
    dados_plotagem.append((classificacao[i][0], tec_mais_longevo[0][1]))
    
for x, y in dados_plotagem:    
    dados_plotagem_x.append(x)
    dados_plotagem_y.append(y)

plt.figure(figsize=(8,6))
plt.title('Campeonato Brasileiro 2019')

plt.scatter(dados_plotagem_x, dados_plotagem_y)
plt.xlabel('Posição')
plt.ylabel('Técnico mais longevo (rodadas)')
plt.xscale('linear')

plt.xticks(dados_plotagem_x)
plt.yticks(dados_plotagem_y)
plt.plot(dados_plotagem_x, dados_plotagem_y, linestyle='--', marker='o', color='red', label='Linha de Conexão')
plt.show()
```

## RECAPITULANDO

Precisávamos responder a seguinte pergunta:
<div style="text-align:center; font-size:20px">
Será que, <b>em geral</b>, times com técnicos mais longevos ficam em uma posição melhor na tabela, no fim do campeonato?    
</div>
<br>
Com isso em mente, e analisando o gráfico, podemos concluir que:

- Mesmo times com técnicos pouco longevos, como o 3º lugar, conseguem se classificar bem no campeonato
- O 1º lugar não teve o técnico mais longevo
- O 12º lugar, mesmo com um dos técnicos mais longevos da campeonato, não teve uma colocação boa
- De fato, alguns times com os técnicos menos longevos do campeonato ficaram nas últimas posições

### Conclusão

Não é possível encontrar uma relação direta entre longevidade do técnico no time e sua classificação. Entendemos que, isso se dá por que futebol é um esporte extremamente estatístico e a qualidade de um time depende de inúmeras variáves e situações, no entanto, neste caso específico, podemos notar que os técnicos mais longevos das 10 melhores colocações jogaram pelo menos metade do campeonato, o que nos sugere que sim, a longevidade influencia positivamente, mas não é determinante para a colocação final do time 
