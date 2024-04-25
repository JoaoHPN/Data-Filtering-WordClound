#Start by installing the libraries.
!pip install pandas
!pip install wordcloud -q

#Some more libraries.
import pandas as pd
import numpy as np
import re
import matplotlib.pyplot as plt
from PIL import Image
from wordcloud import WordCloud, STOPWORDS, ImageColorGenerator

#Read columns in Excel divided by ";"
df = pd.read_csv('/content/GAVMR.csv', sep=';')

#Deleted unnecessary columns.
colunas_para_deletar = ['Criado Por', 'Data de Criação', 'Hora de Criação', 'Atualizado Por', 'Data de Atualização', 'Hora de Atualização', 'Emitentes', 'Áreas de Negócio', 'Gerência', 'Gerência de Área Atendida', 'Data do Evento', 'Hora do Evento', 'Turno', 'País', 'Estado', 'Município', 'Paralisação da Operação', 'Data de Início da Paralisação', 'Hora de Início da Paralisação', 'Data Final da Paralisação', 'Hora Final da Paralisação', 'AVU', 'Ocorrência Associada', 'Responsável Social', 'Responsável Segurança', 'Gerência Social Responsável', 'Gradação', 'Questionamento Saúde & Segurança', 'Questionamento Financeiro | Interrupção Operacional', 'Questionamento Meio Ambiente | Comunidades | Reputacional', 'Utilização de Arma', 'Letalidade da Arma', 'Houve Disparo', 'Participação de Sindicatos', 'Quais Sindicatos', 'Participação de Empresas', 'Quais Empresas', 'Causas', 'Impactos', 'Pessoas Comunicadas', 'Objeto furtado/roubado', 'Repercussão da Mídia?', 'Tipo da Mídia', 'Quais Mídias', 'Tecnologias Empregadas', 'Presença da Segurança Publica', 'Utilização de Tecnologia', 'Prisão', 'Material Recuperado', 'Ocorrência Neutralizada',	'Ferrovia', 'Km da Ferrovia',	'Metros da Ferrovia',	'Crise', 'Pessoas Envolvidas',	'Tratativa', 'Fato Gerador',	'Povo',	'Fator Externo', 'Mobile',	'Ativo',	'Pendente']
df = df.drop(colunas_para_deletar, axis=1)
#df.head()

#Filter specific words in a column and remove texts that have other types of words.
#The "Resumo" column is being used.
palavras_procuradas = [" ", " ", " ", " ", " ", " ", " "]
df_filtrado = df[df['Resumo'].apply(lambda x: any(palavra in x for palavra in palavras_procuradas) and all(palavra not in x.lower() for palavra in [' ', ' ', ' ']))]

#Improve text visualization.
pd.set_option('display.max_colwidth', None)
pd.set_option('display.max_columns', None)
pd.set_option('display.max_rows', None)
print("\nDataFrame filtrado para linhas contendo pelo menos uma das palavras:")
print(df_filtrado["Resumo"])

#Function to extract numbers after the words "post.a.search.here", "post.a.search.here" or "post.a.search.here" with space
def extrair_numeros_km(texto):
    padrao_km = re.compile(r'\b(?: post.a.search.here|post.a.search.here|post.a.search.here)\s*(\d+)\b')
    numeros_km = re.findall(padrao_km, texto)
    return numeros_km

# Apply the function to the 'Resumo' column
df['Numeros_km'] = df['Resumo'].apply(extrair_numeros_km)

# Filter lines where at least one number was found after the words "post.a.search.here", "post.a.search.here" or "post.a.search.here"
filtro_numeros_km = df['Numeros_km'].apply(lambda x: len(x) > 0)
df_filtrado_resumo = df.loc[filtro_numeros_km, 'Numeros_km']

#Display the 'Numeros_km' column of the filtered DataFrame
print(df_filtrado_resumo)

#WordClound
#LOCAL address of YOUR image
rio_mask = np.array(Image.open("LocalAddressOfYourImage.png"))

#Generate a wordcloud
wordcloud = WordCloud(stopwords=stopwords,
                      background_color="black",
                      width=1000, height=1000, max_words=2000,
                      mask=rio_mask, max_font_size=200,
                      min_font_size=1).generate(texto_resumos)

#Show the final image
fig, ax = plt.subplots(figsize=(10,10))
ax.imshow(wordcloud, interpolation='bilinear')
ax.set_axis_off()

plt.imshow(wordcloud)
wordcloud.to_file("LocalAddressOfYourImage.png")


