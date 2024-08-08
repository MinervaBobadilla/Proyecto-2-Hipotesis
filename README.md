

**Ficha Técnica: Proyecto 2 Hipótesis**

**Título del Proyecto**

Hipótesis

**Objetivo**

Obtener una estrategia efectiva para segmentar su base de clientes, utilizando el proceso de Análisis de Datos y desarrollando la metodología de RFM. Identificar características técnicas, para encontrar qué hace que una canción sea más escuchada. Mediante técnica de validación (refutar o confirmar) 

las hipótesis mediante el análisis de los datos, y proporcionar recomendaciones estratégicas basadas en los hallazgos, para que la discográfica y el nuevo artista puedan tomar decisiones informadas que aumenten sus posibilidades de conseguir el “éxito”.

**Equipo**

Trabajo de equipo (B. Morales - M. Bobadilla).

**Herramientas y Tecnologías**

Listado de herramientas y tecnologías utilizadas en el proyecto:

1) Big Query (SQL).

2) Power BI (Python).

3) Workspace Google ( Presentaciones, Gemini y Documentos).

4) Videos Looms.

5) Git Hub.

**Procesamiento y análisis**

**1) Proceso de Análisis de Datos:**

**1.1 Procesar y preparar base de datos**

a) Conectar/importar datos a plataforma Big Query. Tablas importadas: track_in_competition, track_in_spotify y track_technical_info.

b) Identificar y manejar valores nulos, encontrando 50 nulos in shazam charts en la tabla track_in_competition y 95 nulos en key en la tabla

track_technical_info.

c) Identificar y manejar valores duplicados, identificando duplicados en track_in_spotify.

d) Identificar y manejar datos fuera del alcance del análisis, eliminamos las columnas key y mode considerando que no son relevantes en éste análisis.

e) Identificar y manejar datos discrepantes en variables categóricas, realizando el manejo de símbolos en track_in_spotify.

f) Identificar y manejar datos discrepantes en variables numéricas, identificación de discrepancias en valores numéricos utilizando las funciones MIN, MAX y AVG.

g) Comprobar y cambiar tipo de dato de String a Integer.

h) Crear nuevas variables como released_date.

i) Unir tablas mediante Left Join

j) Construir tablas auxiliares con la función WITH para conocer el total de canciones por artista y que conteo tienen en los playlist.


A continuación ejemplos de lo anterior:




```

CREATE TABLE `Dataset.track_in_spotify_limpia` AS

SELECT
  
  track_id,
 
  LOWER(track_name) AS track_name_limpio,
  
  LOWER(artist_s_name) AS artist_s_name_limpio,
 
  artist_count,
  
  released_year,
  
  released_month,
  
  released_day,
  
  spotify_playlists,
  
  spotify_charts,
  
  CAST(streams AS INT64) AS streams_limpio,
  
  DATE(CONCAT(CAST(released_year AS STRING), '-', CAST(released_month AS STRING), '-', CAST(released_day AS STRING))) AS released_date


FROM

  `Dataset.track_in_spotify`


WHERE

  streams NOT LIKE '%BPM%'
  
  AND track_id NOT IN ('3814670', '5080031');


  
```



```


SELECT

C.track_id,

C.track_name_limpio,

C.artist_s_name_limpio,

C.artist_count,

C.spotify_playlists,

C.spotify_charts,

C.streams_limpio,

C.released_year,

C.released_date,

D.in_apple_playlists,

D.in_apple_charts,

D.in_deezer_playlists,

D.in_deezer_charts,

D.in_shazam_charts,

  (C.spotify_playlists + D.in_apple_playlists + D.in_deezer_playlists) AS total_playslists,

E.bpm,

E.danceability,

E.valence,

E.energy,

E.acousticness,

E.instrumentalness,

E.liveness,

E.speechiness


FROM
 `Dataset.track_in_competition_info_limpia` D


LEFT JOIN
 `Dataset.track_in_spotify_limpia` C

 
ON
  C.track_id = D.track_id



LEFT JOIN
 `Dataset.track_in_technical_info_limpia` E

  
ON
  C.track_id = E.track_id


  WHERE
  C.track_id IS NOT NULL'


  
```





**1.2 Análisis exploratorio**


a) Agrupar datos según variables categóricas.

b) Visualizar las variables categóricas.

c) Aplicar medidas de tendencia central.

d) Visualizar distribución.

e) Aplicar medidas de dispersión.

f) Visualizar el comportamiento de los datos a lo largo del tiempo.

g) Calcular cuartiles, deciles o percentiles.

h) Calcular correlación entre variables.


A continuación ejemplos de lo anterior:




```


WITH QUARTILES AS (
    SELECT track_id,
           streams_limpio,
           bpm,
           danceability,
           valence,
           energy,
           acousticness,
           instrumentalness,
           liveness,
           speechiness,
           NTILE(4) OVER (ORDER BY streams_limpio) AS quartile_streams,
           NTILE(4) OVER (ORDER BY bpm) AS quartile_bpm,
           NTILE(4) OVER (ORDER BY danceability) AS quartile_danceability,
           NTILE(4) OVER (ORDER BY valence) AS quartile_valence,
           NTILE(4) OVER (ORDER BY energy) AS quartile_energy,
           NTILE(4) OVER (ORDER BY acousticness) AS quartile_acousticness,
           NTILE(4) OVER (ORDER BY instrumentalness) AS quartile_instrumentalness,
           NTILE(4) OVER (ORDER BY liveness) AS quartile_liveness,
           NTILE(4) OVER (ORDER BY speechiness) AS quartile_speechiness
    FROM `Dataset.track_in_limpia`
)
SELECT
    a.*,
    CASE 
        WHEN QUARTILES.quartile_streams >= 1 THEN 'bajo'
        WHEN QUARTILES.quartile_streams = 2 THEN 'medio bajo'
        WHEN QUARTILES.quartile_streams = 3 THEN 'medio'
        WHEN QUARTILES.quartile_streams = 4 THEN 'alto'
    END AS cat_streams,
    CASE 
        WHEN QUARTILES.quartile_bpm = 1 THEN 'lento'
        WHEN QUARTILES.quartile_bpm = 2 THEN 'moderado'
        WHEN QUARTILES.quartile_bpm = 3 THEN 'rapido'
        WHEN QUARTILES.quartile_bpm = 4 THEN 'muyrapido'
    END AS cat_bpm,
    CASE 
        WHEN QUARTILES.quartile_danceability = 1 THEN 'bajo'
        WHEN QUARTILES.quartile_danceability = 2 THEN 'medio bajo'
        WHEN QUARTILES.quartile_danceability = 3 THEN 'medio'
        WHEN QUARTILES.quartile_danceability = 4 THEN 'alto'
    END AS cat_danceability,
    CASE 
        WHEN QUARTILES.quartile_valence = 1 THEN 'bajo'
        WHEN QUARTILES.quartile_valence = 2 THEN 'medio bajo'
        WHEN QUARTILES.quartile_valence = 3 THEN 'medio'
        WHEN QUARTILES.quartile_valence = 4 THEN 'alto'
    END AS cat_valence,
    CASE 
        WHEN QUARTILES.quartile_energy = 1 THEN 'bajo'
        WHEN QUARTILES.quartile_energy = 2 THEN 'medio bajo'
        WHEN QUARTILES.quartile_energy = 3 THEN 'medio'
        WHEN QUARTILES.quartile_energy = 4 THEN 'alto'
    END AS cat_energy,
    CASE 
        WHEN QUARTILES.quartile_acousticness = 1 THEN 'bajo'
        WHEN QUARTILES.quartile_acousticness = 2 THEN 'medio bajo'
        WHEN QUARTILES.quartile_acousticness = 3 THEN 'medio'
        WHEN QUARTILES.quartile_acousticness = 4 THEN 'alto'
    END AS cat_acousticness,
    CASE 
        WHEN QUARTILES.quartile_instrumentalness = 1 THEN 'bajo'
        WHEN QUARTILES.quartile_instrumentalness = 2 THEN 'medio bajo'
        WHEN QUARTILES.quartile_instrumentalness = 3 THEN 'medio'
        WHEN QUARTILES.quartile_instrumentalness = 4 THEN 'alto'
    END AS cat_instrumentalness,
    CASE 
        WHEN QUARTILES.quartile_liveness = 1 THEN 'bajo'
        WHEN QUARTILES.quartile_liveness = 2 THEN 'medio bajo'
        WHEN QUARTILES.quartile_liveness = 3 THEN 'medio'
        WHEN QUARTILES.quartile_liveness = 4 THEN 'alto'
    END AS cat_liveness,
    CASE 
        WHEN QUARTILES.quartile_speechiness = 1 THEN 'bajo'
        WHEN QUARTILES.quartile_speechiness = 2 THEN 'medio bajo'
        WHEN QUARTILES.quartile_speechiness = 3 THEN 'medio'
        WHEN QUARTILES.quartile_speechiness = 4 THEN 'alto'
    END AS cat_speechiness
FROM `Dataset.track_in_limpia` a
LEFT JOIN QUARTILES
ON a.track_id = QUARTILES.track_id;



  
```






**1.3 Aplicar técnica de análisis**


a) Aplicar segmentación.

b) Validar hipótesis.


A continuación ejemplos de lo anterior:



![image](https://github.com/user-attachments/assets/a9bdc0cc-250f-492c-ae07-941aa8d4877a)








![image](https://github.com/user-attachments/assets/371fba59-e3a8-4629-bcf6-a1e834f4f3a9)








![image](https://github.com/user-attachments/assets/64111fe2-1eec-4f39-9481-6b0cdee3a9c9)









![image](https://github.com/user-attachments/assets/5fa3f000-ffc8-411b-8018-a625d8446045)









![image](https://github.com/user-attachments/assets/d6706e42-4163-4a0d-82db-45a92318a7f5)














**Resultados y Conclusiones**


Durante el desarrollo del proyecto se utilizó el proceso de Análisis de Datos y la metodología de segmentación (Cuartiles). El Periodo analizado fue el año 2023, la discográfica planteó una serie de hipótesis, sobre qué hace que una canción sea más escuchada. Estas hipótesis se detallan a continuación y de esto se puede concluir lo

siguiente:



i) Las canciones con un mayor BPM (Beats Por Minuto) tienen más éxito en términos de cantidad de streams en Spotify.

Se refuta esta hipótesis ya que a mayor BPM no hay mayor cantidad de streams, por lo tanto las canciones de mayor bpm no son las que tienen mas cantidad de streams. Se deduce de gráfica de dispersión BPM VS streams_limpio. Esto también se ve reflejado en el valor de la correlación de Pearson = -0,002 lo cual es un valor alejado del 1 para el ajuste perfecto.



ii) Las canciones más populares en el ranking de Spotify también tienen un comportamiento similar en otras plataformas como Deezer.

Se refuta esta hipótesis ya que, las canciones populares en ranking e Spotify no presentan el mismo comportamiento de éxito en las diferentes plataformas. Ver gráfica track_name_limpio vs suma_plataforma_charts (spotify, deezer, apple y shazam).



iii) La presencia de una canción en un mayor número de playlists se relaciona con un mayor número de streams.

Se refuta la hipótesis ya que no se puede comprobar dicho comportamiento, dado que el contar con un mayor número de playlists por canción no es una relación directamente proporcional al número de streams, esto se verifica en gráfico de track name limpiro VS suma playlists , suma streams.



iv) Los artistas con un mayor número de canciones en Spotify tienen más streams.


Esta hipótesis se refuta con la gráfica de artist\_name\_limpio vs total tracks, artist\_name\_limpio vs suma de streams\_limpio, quien tiene mas canciones no son quienes presentan mayor cantidad de streams ver ejemplo de Ed Sheeran vas sza.



v) Las características de la canción influyen en el éxito en términos de cantidad de streams en Spotify.


Esta hipótesis es refutada ya que las características técnicas como bpm, acousticness, danceability, energy, instumentalness, liveness, speechiness y valence todas presentan un bajo valor en su correlación de Pearson respectivamente, ver gráficos característica técnica VS streams\_limpio excepto la correlación entre total\_playlists vs streams\_limpio que es la

más cercana a 1 con un valor de 0,7835, sin embargo no es una característica técnica.



**Limitaciones/Próximos Pasos**

Sin observaciones.





**Enlaces de interés**


[**Bigquery**](https://console.cloud.google.com/bigquery?project=proyecto-2-hipotesis-17062024&ws=!1m0)


[**PowerBi**](https://drive.google.com/file/d/1ig8KKzgUZN-WlLBDr7E63-PZHYMMtPG-/view?usp=drive_link)[** ](https://drive.google.com/file/d/1ig8KKzgUZN-WlLBDr7E63-PZHYMMtPG-/view?usp=drive_link)[archivo**](https://drive.google.com/file/d/1ig8KKzgUZN-WlLBDr7E63-PZHYMMtPG-/view?usp=drive_link)


[**Power**](https://drive.google.com/file/d/1ws1om_FeAC3lTGDj2if46MP8QsVhcLmj/view?usp=drive_link)[** ](https://drive.google.com/file/d/1ws1om_FeAC3lTGDj2if46MP8QsVhcLmj/view?usp=drive_link)[Bi**](https://drive.google.com/file/d/1ws1om_FeAC3lTGDj2if46MP8QsVhcLmj/view?usp=drive_link)[** ](https://drive.google.com/file/d/1ws1om_FeAC3lTGDj2if46MP8QsVhcLmj/view?usp=drive_link)[PDF**](https://drive.google.com/file/d/1ws1om_FeAC3lTGDj2if46MP8QsVhcLmj/view?usp=drive_link)


[**Presentación**](https://docs.google.com/presentation/d/1K4xuSHIqPjgXnC60qLWCEGogaf-20r8_10RtDrcMZfU/edit?usp=drive_link)[** ](https://docs.google.com/presentation/d/1K4xuSHIqPjgXnC60qLWCEGogaf-20r8_10RtDrcMZfU/edit?usp=drive_link)[archivo**](https://docs.google.com/presentation/d/1K4xuSHIqPjgXnC60qLWCEGogaf-20r8_10RtDrcMZfU/edit?usp=drive_link)


[**Presentación**](https://www.loom.com/share/1de5292285c3485591342e10cda3d72b?sid=8bfde01e-835e-4233-be1a-be97762adb81)[** ](https://www.loom.com/share/1de5292285c3485591342e10cda3d72b?sid=8bfde01e-835e-4233-be1a-be97762adb81)[video**](https://www.loom.com/share/1de5292285c3485591342e10cda3d72b?sid=8bfde01e-835e-4233-be1a-be97762adb81)



<a name="br5"></a> 

