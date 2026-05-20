# telecom_analysis
**Objetivos **
- El objetivo del proyecto es analizar el comportamiento de usuarios de ConnectaTel, una empresa de telecomunicaciones, para:
*Limpiar y preparar los datos detectando valores inválidos, centinelas y fechas erróneas.
*Segmentar clientes por nivel de uso y edad.
*Identificar patrones de consumo entre planes Básico y Premium.
*Generar recomendaciones para optimizar la oferta de planes y aumentar la conversión a Premium.
**reproducir el análisis de principio a fin:**
  # 1. Cargar datos
users = pd.read_csv('users.csv')
usage = pd.read_csv('usage.csv')

# 2. Limpieza
age_mediana = users[users['age'] != -999]['age'].median()
users['age'] = users['age'].replace(-999, age_mediana)
users['city'] = users['city'].replace('?', pd.NA)
users['reg_date'] = pd.to_datetime(users['reg_date'], errors='coerce')
users.loc[users['reg_date'].dt.year > 2024, 'reg_date'] = pd.NaT

# 3. Agregar usage
usage['is_text'] = (usage['type'] == 'text').astype(int)
usage['is_call'] = (usage['type'] == 'call').astype(int)
usage_agg = usage.groupby('user_id').agg(
    cant_mensajes        = ('is_text',  'sum'),
    cant_llamadas        = ('is_call',  'sum'),
    cant_minutos_llamada = ('duration', 'sum')
).reset_index()

# 4. Merge
user_profile = users.merge(usage_agg, on='user_id', how='left')

# 5. Segmentación
user_profile['grupo_uso'] = np.select(
    [(user_profile['cant_llamadas'] < 5)  & (user_profile['cant_mensajes'] < 5),
     (user_profile['cant_llamadas'] < 10) & (user_profile['cant_mensajes'] < 10)],
    ['Bajo uso', 'Uso medio'], default='Alto uso'
)
user_profile['grupo_edad'] = np.select(
    [user_profile['age'] < 30, user_profile['age'] < 60],
    ['Joven', 'Adulto'], default='Adulto Mayor'
)

# 6. Outliers
user_profile['cant_minutos_llamada'] = user_profile['cant_minutos_llamada'].clip(upper=61.86)
