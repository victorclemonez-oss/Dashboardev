import streamlit as st
import pandas as pd
import plotly.express as px
import plotly.graph_objects as go

# --- CONFIGURAÇÃO DA PÁGINA ---
st.set_page_config(layout="wide", page_title="Escola de Vendas | Dashboard")

# --- ESTILIZAÇÃO CSS (TEMA FUTURISTA) ---
st.markdown("""
    <style>
    .main { background-color: #FFFFFF; }
    [data-testid="stSidebar"] { 
        background-color: #001E3C; 
        border-right: 5px solid #FF6700;
    }
    .stMetric {
        background: #FFFFFF;
        padding: 20px;
        border-radius: 15px;
        box-shadow: 0 4px 20px rgba(0,0,0,0.05);
        border-bottom: 4px solid #FF6700;
    }
    h1, h2, h3 { color: #001E3C; font-family: 'Inter', sans-serif; }
    .stTabs [data-baseweb="tab-list"] { gap: 24px; }
    .stTabs [data-baseweb="tab"] {
        height: 50px;
        white-space: pre-wrap;
        background-color: #F0F2F6;
        border-radius: 10px 10px 0 0;
        gap: 1px;
        padding-top: 10px;
    }
    .stTabs [aria-selected="true"] { background-color: #FF6700; color: white; }
    </style>
    """, unsafe_allow_html=True)

# --- FUNÇÃO PARA PROCESSAR O ARQUIVO DE TREINAMENTOS ---
def process_training_data(file):
    df = pd.read_excel(file)
    # Limpeza da Pontuação (ex: "8 / 10" -> 8)
    df['Nota_Num'] = df['Pontuacao'].str.split('/').str[0].astype(float)
    return df

# --- SIDEBAR ---
with st.sidebar:
    st.markdown("<h1 style='color: white;'>Escola de Vendas</h1>", unsafe_allow_html=True)
    st.markdown("<p style='color: #ADB5BD;'>Sistema de Gestão de Performance</p>", unsafe_allow_html=True)
    st.divider()
    page = st.radio("Navegação", ["📊 Dashboard Principal", "📁 Adicionar Arquivos"])

# --- ABA 2: ADICIONAR ARQUIVOS (Sua solicitação) ---
if page == "📁 Adicionar Arquivos":
    st.title("📁 Gerenciamento de Dados")
    st.subheader("Suba seus novos arquivos do Drive aqui")
    
    upload_train = st.file_uploader("Arquivo de Treinamentos (Excel)", type=["xlsx"])
    upload_sales = st.file_uploader("Arquivo de Vendas/Crescimento (Excel)", type=["xlsx"])
    upload_ops = st.file_uploader("Arquivo de OPS (Excel)", type=["xlsx"])
    
    if upload_train:
        st.success("Arquivo de Treinamento carregado com sucesso!")
        # Aqui você pode salvar em um banco de dados ou cache
    st.info("Nota: Para persistência de longo prazo, conectaremos este app ao seu Google Sheets.")

# --- ABA 1: DASHBOARD PRINCIPAL ---
else:
    # Header com Filtro no lado direito
    head_left, head_right = st.columns([3, 1])
    with head_left:
        st.title("🚀 Performance Geral")
    with head_right:
        filtro_geral = st.selectbox("Filtrar por GD ou Filial", ["Geral", "EDUARDO", "KATHLYN", "FELLIPE", "ERIKA", "DANIEL", "JADILSON"])

    # Simulação de dados baseada no arquivo que você enviou
    # (Em um caso real, os dados viriam do uploader acima)
    data_mock = {
        'GD': ['EDUARDO', 'KATHLYN', 'FELLIPE', 'ERIKA', 'DANIEL', 'JADILSON'],
        'Participantes': [15, 28, 22, 18, 30, 12],
        'Media_Nota': [9.2, 9.8, 8.9, 9.0, 9.6, 8.5],
        'Colaboradores': [45, 82, 65, 50, 90, 38]
    }
    df_resumo = pd.DataFrame(data_mock)

    # --- LINHA 1: KPIs ---
    kpi1, kpi2, kpi3, kpi4 = st.columns(4)
    with kpi1:
        st.metric("Média Nota Geral", "9.1", "↑ 0.5")
    with kpi2:
        st.metric("Total Participantes", "125", "Ativos")
    with kpi3:
        st.metric("% Crescimento", "14.2%", "↑ 2%")
    with kpi4:
        st.metric("OPS Batidas", "88%", "Meta 90%")

    st.markdown("---")

    # --- LINHA 2: GRÁFICOS DE TREINAMENTO ---
    col1, col2 = st.columns(2)

    with col1:
        st.subheader("📊 Participantes por GD")
        fig_gd = px.bar(df_resumo, x='GD', y='Participantes', 
                         color_discrete_sequence=['#001E3C'], 
                         text_auto=True)
        fig_gd.update_traces(marker_line_color='#FF6700', marker_line_width=1.5)
        st.plotly_chart(fig_gd, use_container_width=True)

    with col2:
        st.subheader("🔝 Filiais com mais Colaboradores")
        # Gráfico de Rosca (Donut) para um ar mais moderno
        fig_filial = px.pie(df_resumo, values='Colaboradores', names='GD', 
                             hole=0.5, color_discrete_sequence=px.colors.sequential.Blues_r)
        st.plotly_chart(fig_filial, use_container_width=True)

    # --- LINHA 3: VENDAS E CRESCIMENTO (Futuro Arquivo) ---
    st.markdown("---")
    col3, col4 = st.columns([2, 1])

    with col3:
        st.subheader("📈 Aumento de Vendas Diárias")
        # Simulação de gráfico de área
        vendas_diarias = pd.DataFrame({'Dia': range(1, 31), 'Vendas': [i**1.2 for i in range(1, 31)]})
        fig_vendas = px.area(vendas_diarias, x='Dia', y='Vendas', 
                              color_discrete_sequence=['#FF6700'])
        st.plotly_chart(fig_vendas, use_container_width=True)

    with col4:
        st.subheader("📦 Crescimento por Categoria")
        # Gráfico de barras horizontal
        cat_data = {'Cat': ['Genéricos', 'Similares', 'OTC'], 'Cresc': [20, 15, 10]}
        fig_cat = px.bar(cat_data, y='Cat', x='Cresc', orientation='h', 
                          color_discrete_sequence=['#001E3C'])
        st.plotly_chart(fig_cat, use_container_width=True)
