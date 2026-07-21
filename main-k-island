import re

import pandas as pd
import streamlit as st
import plotly.graph_objects as go

st.set_page_config(page_title="우리나라 섬 인구 피라미드", layout="wide")

POPULATION_URL = "https://raw.githubusercontent.com/greatsong/modudata/main/data/population_yearly.csv.gz"
TARGET_YEAR = 2026

# 섬(도서) 지역으로 알려진 시군구 / 시도 목록입니다.
# - 제주도는 제주시·서귀포시 두 시로 나뉘어 있어서 '시도' 기준으로 묶었습니다.
# - 나머지는 섬이 대부분(또는 전부)인 시·군 이름입니다.
ISLAND_REGIONS = {
    "제주도 (제주시 · 서귀포시)": {"기준": "시도", "값": "제주특별자치도"},
    "거제도 (경남 거제시)": {"기준": "시군구", "값": "거제시"},
    "남해도 (경남 남해군)": {"기준": "시군구", "값": "남해군"},
    "울릉도 (경북 울릉군)": {"기준": "시군구", "값": "울릉군"},
    "강화도 (인천 강화군)": {"기준": "시군구", "값": "강화군"},
    "옹진군 섬들 - 백령도 · 연평도 등 (인천)": {"기준": "시군구", "값": "옹진군"},
    "완도군 섬들 - 노화도 · 보길도 등 (전남)": {"기준": "시군구", "값": "완도군"},
    "진도군 섬들 (전남)": {"기준": "시군구", "값": "진도군"},
    "신안군 섬들 - 임자도 · 흑산도 등 (전남)": {"기준": "시군구", "값": "신안군"},
}

AGE_BIN_SIZE = 10
AGE_MAX = 100  # '100세 이상'까지 포함


@st.cache_data
def load_population():
    df = pd.read_csv(POPULATION_URL, compression="gzip", dtype={"코드": str})
    year_str = df["연도"].astype(str).str.extract(r"(\d{4})")[0]
    df = df[year_str == str(TARGET_YEAR)].copy()
    return df


def make_age_bins():
    """0~9세, 10~19세, ... 90세 이상 형태의 나이대 구간을 만듭니다."""
    bins = []
    start = 0
    while start < AGE_MAX:
        end = min(start + AGE_BIN_SIZE - 1, AGE_MAX - 1)
        if start >= 90:
            label = "90세 이상"
            ages = list(range(90, AGE_MAX)) + ["100세 이상"]
            bins.append((label, ages))
            break
        else:
            label = f"{start}~{end}세"
            ages = list(range(start, end + 1))
            bins.append((label, ages))
        start += AGE_BIN_SIZE
    return bins


def age_col(gender_prefix, age):
    if age == "100세 이상":
        return f"{gender_prefix}_100세 이상"
    return f"{gender_prefix}_{age}세"


def build_pyramid_table(rows: pd.DataFrame):
    bins = make_age_bins()
    records = []
    for label, ages in bins:
        male_cols = [age_col("남", a) for a in ages]
        female_cols = [age_col("여", a) for a in ages]
        male_sum = rows[male_cols].sum().sum()
        female_sum = rows[female_cols].sum().sum()
        records.append({"나이대": label, "남": int(male_sum), "여": int(female_sum)})
    return pd.DataFrame(records)


st.title("우리나라 섬 인구 피라미드")
st.caption(f"{TARGET_YEAR}년 6월 기준 · 성별 · 나이대별 인구")

df = load_population()

region_name = st.selectbox("섬 지역을 선택하세요", list(ISLAND_REGIONS.keys()))
region = ISLAND_REGIONS[region_name]

if region["기준"] == "시도":
    region_df = df[df["시도"] == region["값"]].copy()
else:
    region_df = df[df["시군구"] == region["값"]].copy()

st.subheader("이 지역에 속한 섬(읍·면·동) 목록")
st.caption("행정구역 이름이 실제 섬 이름과 같은 경우가 많아요. (예: 노화읍 → 노화도)")

total_cols = [c for c in df.columns if c.startswith("계_")]
region_df["인구"] = region_df[total_cols].sum(axis=1)

island_list = (
    region_df.groupby("동", as_index=False)["인구"]
    .sum()
    .sort_values("인구", ascending=False)
    .reset_index(drop=True)
)
st.dataframe(island_list, use_container_width=True, hide_index=True)

selected_islands = st.multiselect(
    "그래프로 볼 섬(읍·면·동)을 골라보세요. 아무것도 고르지 않으면 전체를 합산해서 보여줘요.",
    options=island_list["동"].tolist(),
)

if selected_islands:
    rows = region_df[region_df["동"].isin(selected_islands)]
    title_suffix = ", ".join(selected_islands)
else:
    rows = region_df
    title_suffix = "전체"

pyramid_df = build_pyramid_table(rows)

fig = go.Figure()
fig.add_trace(
    go.Bar(
        y=pyramid_df["나이대"],
        x=-pyramid_df["남"],
        name="남",
        orientation="h",
        marker_color="#378ADD",
        customdata=pyramid_df["남"],
        hovertemplate="%{y} 남: %{customdata:,}명<extra></extra>",
    )
)
fig.add_trace(
    go.Bar(
        y=pyramid_df["나이대"],
        x=pyramid_df["여"],
        name="여",
        orientation="h",
        marker_color="#D4537E",
        hovertemplate="%{y} 여: %{x:,}명<extra></extra>",
    )
)

max_val = max(pyramid_df["남"].max(), pyramid_df["여"].max())
fig.update_layout(
    title=f"{region_name} — {title_suffix}",
    barmode="relative",
    bargap=0.1,
    xaxis=dict(
        title="인구수(명)",
        tickvals=[-max_val, -max_val / 2, 0, max_val / 2, max_val],
        ticktext=[
            f"{int(max_val):,}",
            f"{int(max_val/2):,}",
            "0",
            f"{int(max_val/2):,}",
            f"{int(max_val):,}",
        ],
    ),
    yaxis=dict(title="나이대"),
    height=600,
)

st.plotly_chart(fig, use_container_width=True)

total_male = int(pyramid_df["남"].sum())
total_female = int(pyramid_df["여"].sum())
col1, col2, col3 = st.columns(3)
col1.metric("남자 인구", f"{total_male:,}명")
col2.metric("여자 인구", f"{total_female:,}명")
col3.metric("전체 인구", f"{total_male + total_female:,}명")
