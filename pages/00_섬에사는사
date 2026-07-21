import pandas as pd
import streamlit as st
import plotly.express as px

from islands_data import ISLAND_REGIONS, TARGET_YEAR, load_population, total_pop_columns, region_dataframe

st.set_page_config(page_title="섬 인구 순위", layout="wide")

st.title("가장 사람이 많이 사는 섬 순위")
st.caption(f"{TARGET_YEAR}년 6월 기준 · 읍·면·동 단위 인구")
st.info(
    "여기서 '섬 이름'은 실제 지도상의 섬 이름이 아니라, 그 섬이 속한 행정구역(읍·면·동) 이름이에요. "
    "노화읍 · 자은면처럼 대부분 ~읍 또는 ~면으로 표기되고, 일부는 ~동으로 표기됩니다. "
    "행정구역 이름이 실제 섬 이름과 같은 경우가 많지만(예: 노화읍 → 노화도), 100% 같지는 않을 수 있어요."
)

df = load_population()
total_cols = total_pop_columns(df)

# 9개 섬 지역을 모두 합쳐서, 그 안의 모든 읍·면·동(섬)을 한 번에 비교합니다.
rows = []
for region_name, region in ISLAND_REGIONS.items():
    region_df = region_dataframe(df, region)
    region_df["인구"] = region_df[total_cols].sum(axis=1)
    grouped = region_df.groupby("동", as_index=False)["인구"].sum()
    grouped["소속 지역"] = region_name
    grouped["소속 지역 면적(km²)"] = region["면적_km2"]
    rows.append(grouped)

all_islands = (
    pd.concat(rows, ignore_index=True)
    .sort_values("인구", ascending=False)
    .reset_index(drop=True)
)
all_islands.insert(0, "순위", all_islands.index + 1)
all_islands = all_islands.rename(columns={"동": "섬(읍·면·동)"})

top_n = st.slider("몇 위까지 볼까요?", min_value=5, max_value=len(all_islands), value=15, step=5)
top_df = all_islands.head(top_n)

fig = px.bar(
    top_df.sort_values("인구"),
    x="인구",
    y="섬(읍·면·동)",
    orientation="h",
    color="소속 지역",
    hover_data={"소속 지역 면적(km²)": True, "인구": ":,"},
    title=f"인구가 많은 섬(읍·면·동) TOP {top_n}",
)
fig.update_layout(height=max(400, top_n * 30), yaxis=dict(title=""), xaxis=dict(title="인구수(명)"))

st.plotly_chart(fig, use_container_width=True)

st.subheader("전체 순위표")
st.caption(
    "면적은 섬(읍·면·동) 하나하나가 아니라, 그 섬이 속한 시·군 전체의 면적입니다. "
    "정확한 개별 섬 면적 데이터는 아직 이 앱에 없어요."
)
st.dataframe(
    all_islands[["순위", "섬(읍·면·동)", "소속 지역", "인구", "소속 지역 면적(km²)"]],
    use_container_width=True,
    hide_index=True,
)

st.divider()
st.page_link("main.py", label="← 인구 피라미드 페이지로 돌아가기", icon="🏠")
