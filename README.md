# An-lise-de-Consumo-de-Energia-no-Brasil-por-Esta-o-do-Ano

#Explicação: Minha primeira ideia foi analisar o HTML da API como forma de base para meu projeto, essa ideia é possivel, 
#porem, perde-se muito do desempenho do programa, com possibilidade de erro. Então,
#utilizei uma base de dados da ONS para tentar simular e chegar
#proximo do resultado da API. Com o passar do codigo, irei explicar passo a passo.


import pandas as pd #Aumentar a manipulação dos dados
import requests     #Usado para baixar os dados de Excel de URL                     
import matplotlib.pyplot as plt     #Criar tabelas e graficos

#uma def para ler a url do arquivo de Excel
def load_data_from_url(url):
    response = requests.get(url)
    if response.status_code == 200:
        return pd.read_excel(response.content)
    else:
        print("Erro ao carregar os dados da URL:", url)
        return None

#URL dos arquivos de Excel, que será util para conseguirmos usar os dados
urls = [
    'https://ons-aws-prod-opendata.s3.amazonaws.com/dataset/cmo_tm/CMO_SEMIHORARIO_2021.xlsx',
    'https://ons-aws-prod-opendata.s3.amazonaws.com/dataset/cmo_tm/CMO_SEMIHORARIO_2022.xlsx', 
    'https://ons-aws-prod-opendata.s3.amazonaws.com/dataset/cmo_tm/CMO_SEMIHORARIO_2023.xlsx'
]

dfs = [load_data_from_url(url) for url in urls] #Unir os arquivos, para ficar mais facil achar os dados
df_final = pd.concat(dfs, ignore_index=True)

#Definimos a data de inicio de forma mais geral
estacoes_ano = {
    "Primavera": ("09-23", "12-21"),
    "Verão": ("12-22", "03-19"),
    "Outono": ("03-20", "06-20"),
    "Inverno": ("06-21", "09-22")
}

#Definimos uma lista então, para que podermos armazenar val_cargaglobal
medias_por_estacao = []

#Fazemos um loop para achar os valores 'val_cargaglobal' dos anos de 2021, 2022 e 2023 e colocamos eles em uma lista
for ano in range(2021, 2024):
   
    #Fazemos um loop para achar os valores
    for estacao, (inicio, fim) in estacoes_ano.items():
        inicio = pd.to_datetime(f"{ano}-{inicio}", format='%Y-%m-%d')
        fim = pd.to_datetime(f"{ano}-{fim}", format='%Y-%m-%d')
        
        #Filtramos as datas dentro dos urls
        filtered_df = df_final[(df_final['dat_referencia'] >= inicio) & 
                               (df_final['dat_referencia'] <= fim)]
        
        #Calculamos a media dos 'val_cargaglobal' dentro daquela estacção
        media_carga_global = filtered_df['val_cargaglobal'].mean()
        
        # Adicionar as médias à lista
        medias_por_estacao.append({
            "Ano": ano,
            "Estação do Ano": estacao,
            "Média da Carga Global": media_carga_global
        })

#Criamos um DataFrame para exibir os 'val_cargaglobal'
df_medias_por_estacao = pd.DataFrame(medias_por_estacao)

#Vamos agora achar os valores minimos e maximos dessas medias
menores_medias = df_resultados.groupby('Estação do Ano')['Média da Carga Global'].min()
maiores_medias = df_resultados.groupby('Estação do Ano')['Média da Carga Global'].max()

#Printamos as maiores e menores medias
print("Menores médias da carga global por estação do ano:" + str(menores_medias))
print("\nMaiores médias da carga global por estação do ano:" + str(maiores_medias))

# Plotar o gráfico de barras
plt.figure(figsize=(10, 6))
for estacao in df_medias_por_estacao['Estação do Ano'].unique():
    df_temp = df_medias_por_estacao[df_medias_por_estacao['Estação do Ano'] == estacao]
    plt.bar(df_temp['Ano'], df_temp['Média da Carga Global'], label=estacao)

plt.xlabel('Ano')
plt.ylabel('Média da Carga Global')
plt.title('Média da Carga Global por Estação do Ano (2021-2023)')
plt.legend()
plt.grid(True)
plt.show()

#Novamente, essa é uma forma mais rapida de tentar resolver esse problema, passando pelos pontos pedidos
#Como dito anteriormente, é possivel usar o URL do API, ou criar uma planilha do excel com todos os dados do
#API, mas desta forma que é mais comparativa e mostrando os codigos e a logica de programação,
#provamos que é possivel fazer e achar os resultados

#Gerenciamento de Dependências: import matplotlib.pyplot as plt; import requests; import pandas as pd
