import itertools
import math
from typing import Dict, List, Set, Tuple

import pandas as pd
import streamlit as st

st.set_page_config(page_title="롤 내전 팀 생성기", layout="wide")

LANES = ["TOP", "JGL", "MID", "ADC", "SUP"]
TIER_SCORES = {
    "IRON": 1,
    "BRONZE": 2,
    "SILVER": 3,
    "GOLD": 4,
    "PLATINUM": 5,
    "EMERALD": 6,
    "DIAMOND": 7,
    "MASTER": 8,
    "GRANDMASTER": 9,
    "CHALLENGER": 10,
}

DEFAULT_PLAYERS = [
    {"name": "Player1", "tier": "GOLD", "top": True, "jgl": False, "mid": True, "adc": False, "sup": False},
    {"name": "Player2", "tier": "GOLD", "top": False, "jgl": True, "mid": False, "adc": False, "sup": True},
    {"name": "Player3", "tier": "PLATINUM", "top": False, "jgl": False, "mid": True, "adc": True, "sup": False},
    {"name": "Player4", "tier": "SILVER", "top": True, "jgl": False, "mid": False, "adc": False, "sup": True},
    {"name": "Player5", "tier": "GOLD", "top": False, "jgl": True, "mid": False, "adc": True, "sup": False},
    {"name": "Player6", "tier": "PLATINUM", "top": True, "jgl": False, "mid": False, "adc": False, "sup": False},
    {"name": "Player7", "tier": "GOLD", "top": False, "jgl": True, "mid": False, "adc": False, "sup": False},
    {"name": "Player8", "tier": "SILVER", "top": False, "jgl": False, "mid": True, "adc": False, "sup": True},
    {"name": "Player9", "tier": "GOLD", "top": False, "jgl": False, "mid": False, "adc": True, "sup": False},
    {"name": "Player10", "tier": "PLATINUM", "top": False, "jgl": False, "mid": False, "adc": False, "sup": True},
]


def init_state():
    if "players_df" not in st.session_state:
        st.session_state.players_df = pd.DataFrame(DEFAULT_PLAYERS)
    if "duo_pairs" not in st.session_state:
        st.session_state.duo_pairs = []
    if "enemy_pairs" not in st.session_state:
        st.session_state.enemy_pairs = []


init_state()


def normalize_players_df(df: pd.DataFrame) -> pd.DataFrame:
    expected_cols = ["name", "tier", "top", "jgl", "mid", "adc", "sup"]
    out = df.copy()
    for col in expected_cols:
        if col not in out.columns:
            out[col] = False if col in ["top", "jgl", "mid", "adc", "sup"] else ""

    out = out[expected_cols]
    out["name"] = out["name"].astype(str).str.strip()
    out["tier"] = out["tier"].astype(str).str.upper().str.strip()

    for col in ["top", "jgl", "mid", "adc", "sup"]:
        out[col] = out[col].fillna(False).astype(bool)

    return out



def build_player_dicts(df: pd.DataFrame) -> List[Dict]:
    players = []
    for _, row in df.iterrows():
        lanes = set()
        if row["top"]:
            lanes.add("TOP")
        if row["jgl"]:
            lanes.add("JGL")
        if row["mid"]:
            lanes.add("MID")
        if row["adc"]:
            lanes.add("ADC")
        if row["sup"]:
            lanes.add("SUP")

        players.append(
            {
                "name": row["name"],
                "tier": row["tier"],
                "score": TIER_SCORES.get(row["tier"], 0),
                "lanes": lanes,
            }
        )
    return players



def validate_players(players: List[Dict]) -> List[str]:
    errors = []
    if len(players) != 10:
        errors.append("플레이어는 정확히 10명이어야 합니다.")

    names = [p["name"] for p in players]
    if any(name == "" for name in names):
        errors.append("빈 닉네임이 있습니다.")
    if len(names) != len(set(names)):
        errors.append("닉네임이 중복되었습니다. 모든 닉네임은 고유해야 합니다.")

    invalid_tiers = [p["name"] for p in players if p["score"] == 0]
    if invalid_tiers:
        errors.append(f"유효하지 않은 티어가 있습니다: {', '.join(invalid_tiers)}")

    no_lane_players = [p["name"] for p in players if len(p["lanes"]) == 0]
    if no_lane_players:
        errors.append(f"가능 라인이 하나도 없는 플레이어가 있습니다: {', '.join(no_lane_players)}")

    for lane in LANES:
        possible = [p["name"] for p in players if lane in p["lanes"]]
        if len(possible) < 2:
            errors.append(f"{lane} 가능 인원이 2명 미만이라 두 팀 배정이 불가능합니다.")

    return errors



def pairs_valid(pairs: List[Tuple[str, str]], valid_names: Set[str]) -> List[str]:
    errors = []
    seen = set()
    for a, b in pairs:
        if not a or not b:
            continue
        if a == b:
            errors.append(f"같은 사람끼리 짝지을 수 없습니다: {a}")
            continue
        if a not in valid_names or b not in valid_names:
            errors.append(f"존재하지 않는 플레이어가 pair에 포함되어 있습니다: ({a}, {b})")
            continue
        key = tuple(sorted([a, b]))
        if key in seen:
            errors.append(f"중복 pair가 있습니다: {key[0]} - {key[1]}")
        seen.add(key)
    return errors



def generate_lane_assignments(team_players: List[Dict]) -> List[Dict[str, str]]:
    results = []
    for perm in itertools.permutations(team_players, 5):
        ok = True
        assignment = {}
        for lane, player in zip(LANES, perm):
            if lane not in player["lanes"]:
                ok = False
                break
            assignment[lane] = player["name"]
        if ok:
            results.append(assignment)
    return results



def team_score(team_players: List[Dict]) -> int:
    return sum(p["score"] for p in team_players)



def count_offroles(assignment: Dict[str, str], player_map: Dict[str, Dict]) -> int:
    # 현재 버전에서는 가능 라인만 체크하므로 offrole 개념은 약하게 처리.
    # 여러 라인이 가능한 사람이 배정된 것은 불이익 없음.
    return 0



def violates_duo(team_names: Set[str], duo_pairs: List[Tuple[str, str]]) -> bool:
    for a, b in duo_pairs:
        if (a in team_names) ^ (b in team_names):
            return True
    return False



def violates_enemy(team_names: Set[str], enemy_pairs: List[Tuple[str, str]]) -> bool:
    for a, b in enemy_pairs:
        if a in team_names and b in team_names:
            return True
    return False



def solve_best_match(players: List[Dict], duo_pairs: List[Tuple[str, str]], enemy_pairs: List[Tuple[str, str]]):
    player_map = {p["name"]: p for p in players}
    all_names = [p["name"] for p in players]
    best = None

    for team1_names_tuple in itertools.combinations(all_names, 5):
        team1_names = set(team1_names_tuple)
        team2_names = set(all_names) - team1_names

        if violates_duo(team1_names, duo_pairs):
            continue
        if violates_enemy(team1_names, enemy_pairs):
            continue
        if violates_enemy(team2_names, enemy_pairs):
            continue

        team1_players = [player_map[name] for name in team1_names]
        team2_players = [player_map[name] for name in team2_names]

        team1_assignments = generate_lane_assignments(team1_players)
        if not team1_assignments:
            continue
        team2_assignments = generate_lane_assignments(team2_players)
        if not team2_assignments:
            continue

        score_diff = abs(team_score(team1_players) - team_score(team2_players))

        # 라인 배치가 되는 경우 중 첫 해를 사용하되,
        # 필요하면 여기에 선호 라인/주라인 가중치 확장 가능.
        candidate = {
            "team1_names": sorted(team1_names),
            "team2_names": sorted(team2_names),
            "team1_assignment": team1_assignments[0],
            "team2_assignment": team2_assignments[0],
            "team1_score": team_score(team1_players),
            "team2_score": team_score(team2_players),
            "score_diff": score_diff,
        }

        if best is None or candidate["score_diff"] < best["score_diff"]:
            best = candidate

    return best



def render_assignment(title: str, assignment: Dict[str, str], player_map: Dict[str, Dict], total_score: int):
    st.subheader(title)
    rows = []
    for lane in LANES:
        name = assignment[lane]
        p = player_map[name]
        rows.append(
            {
                "라인": lane,
                "플레이어": name,
                "티어": p["tier"],
                "점수": p["score"],
                "가능 라인": ", ".join(sorted(p["lanes"])),
            }
        )
    st.dataframe(pd.DataFrame(rows), use_container_width=True, hide_index=True)
    st.metric("팀 총점", total_score)


st.title("롤 내전 팀 생성기")
st.caption("티어, 가능한 라인, 묶밸(같은 팀/반대 팀)을 반영해서 5:5 팀을 자동 생성합니다.")

with st.expander("티어 점수 기준", expanded=False):
    tier_df = pd.DataFrame(
        [{"tier": k, "score": v} for k, v in TIER_SCORES.items()]
    )
    st.dataframe(tier_df, use_container_width=True, hide_index=True)
    st.write("점수는 자유롭게 바꾸고 싶다면 코드의 TIER_SCORES 딕셔너리를 수정하면 됩니다.")

left, right = st.columns([1.6, 1])

with left:
    st.subheader("1) 플레이어 입력")
    edited_df = st.data_editor(
        normalize_players_df(st.session_state.players_df),
        num_rows="fixed",
        use_container_width=True,
        column_config={
            "name": st.column_config.TextColumn("닉네임", required=True),
            "tier": st.column_config.SelectboxColumn("티어", options=list(TIER_SCORES.keys()), required=True),
            "top": st.column_config.CheckboxColumn("TOP"),
            "jgl": st.column_config.CheckboxColumn("JGL"),
            "mid": st.column_config.CheckboxColumn("MID"),
            "adc": st.column_config.CheckboxColumn("ADC"),
            "sup": st.column_config.CheckboxColumn("SUP"),
        },
        hide_index=True,
        key="players_editor",
    )
    st.session_state.players_df = normalize_players_df(edited_df)

with right:
    st.subheader("2) 묶밸 설정")
    player_names = [n for n in st.session_state.players_df["name"].tolist() if str(n).strip()]

    st.markdown("**같은 팀으로 묶기 (듀오/고정밸)**")
    duo_count = st.number_input("듀오 개수", min_value=0, max_value=10, value=len(st.session_state.duo_pairs), step=1)
    duo_pairs = []
    for i in range(duo_count):
        c1, c2 = st.columns(2)
        default_a = st.session_state.duo_pairs[i][0] if i < len(st.session_state.duo_pairs) and st.session_state.duo_pairs[i][0] in player_names else None
        default_b = st.session_state.duo_pairs[i][1] if i < len(st.session_state.duo_pairs) and st.session_state.duo_pairs[i][1] in player_names else None
        with c1:
            a = st.selectbox(f"듀오 {i+1} - A", options=player_names, index=player_names.index(default_a) if default_a in player_names else None, key=f"duo_a_{i}") if player_names else None
        with c2:
            b = st.selectbox(f"듀오 {i+1} - B", options=player_names, index=player_names.index(default_b) if default_b in player_names else None, key=f"duo_b_{i}") if player_names else None
        if a and b:
            duo_pairs.append((a, b))
    st.session_state.duo_pairs = duo_pairs

    st.markdown("**반대 팀으로 분리하기**")
    enemy_count = st.number_input("분리 pair 개수", min_value=0, max_value=10, value=len(st.session_state.enemy_pairs), step=1)
    enemy_pairs = []
    for i in range(enemy_count):
        c1, c2 = st.columns(2)
        default_a = st.session_state.enemy_pairs[i][0] if i < len(st.session_state.enemy_pairs) and st.session_state.enemy_pairs[i][0] in player_names else None
        default_b = st.session_state.enemy_pairs[i][1] if i < len(st.session_state.enemy_pairs) and st.session_state.enemy_pairs[i][1] in player_names else None
        with c1:
            a = st.selectbox(f"분리 {i+1} - A", options=player_names, index=player_names.index(default_a) if default_a in player_names else None, key=f"enemy_a_{i}") if player_names else None
        with c2:
            b = st.selectbox(f"분리 {i+1} - B", options=player_names, index=player_names.index(default_b) if default_b in player_names else None, key=f"enemy_b_{i}") if player_names else None
        if a and b:
            enemy_pairs.append((a, b))
    st.session_state.enemy_pairs = enemy_pairs

st.divider()

players = build_player_dicts(st.session_state.players_df)
player_map = {p["name"]: p for p in players}
errors = validate_players(players)
errors += pairs_valid(st.session_state.duo_pairs, set(player_map.keys()))
errors += pairs_valid(st.session_state.enemy_pairs, set(player_map.keys()))

conflicts = []
for pair in st.session_state.duo_pairs:
    if tuple(sorted(pair)) in {tuple(sorted(x)) for x in st.session_state.enemy_pairs}:
        conflicts.append(pair)
if conflicts:
    errors.append("같은 pair가 '같은 팀'과 '분리'에 동시에 들어가 있습니다.")

col_a, col_b = st.columns([1, 1])
with col_a:
    if st.button("팀 생성", type="primary", use_container_width=True):
        st.session_state["run_solver"] = True
with col_b:
    if st.button("예시 데이터로 초기화", use_container_width=True):
        st.session_state.players_df = pd.DataFrame(DEFAULT_PLAYERS)
        st.session_state.duo_pairs = []
        st.session_state.enemy_pairs = []
        st.rerun()

if errors:
    for err in errors:
        st.error(err)
else:
    if st.session_state.get("run_solver", False):
        result = solve_best_match(players, st.session_state.duo_pairs, st.session_state.enemy_pairs)
        if result is None:
            st.warning("주어진 조건으로는 팀을 만들 수 없습니다. 라인 가능 범위나 묶밸 조건을 조금 완화해보세요.")
        else:
            st.success(f"팀 생성 완료! 두 팀 점수 차이: {result['score_diff']}")
            c1, c2 = st.columns(2)
            with c1:
                render_assignment("팀 1", result["team1_assignment"], player_map, result["team1_score"])
            with c2:
                render_assignment("팀 2", result["team2_assignment"], player_map, result["team2_score"])

            st.subheader("요약")
            summary_df = pd.DataFrame(
                [
                    {"항목": "팀1 총점", "값": result["team1_score"]},
                    {"항목": "팀2 총점", "값": result["team2_score"]},
                    {"항목": "점수 차이", "값": result["score_diff"]},
                    {"항목": "같은 팀 묶기", "값": ", ".join([f"{a}-{b}" for a, b in st.session_state.duo_pairs]) or "없음"},
                    {"항목": "반대 팀 분리", "값": ", ".join([f"{a}-{b}" for a, b in st.session_state.enemy_pairs]) or "없음"},
                ]
            )
            st.dataframe(summary_df, use_container_width=True, hide_index=True)

            csv_rows = []
            for team_label, assignment in [("TEAM1", result["team1_assignment"]), ("TEAM2", result["team2_assignment"])]:
                for lane in LANES:
                    name = assignment[lane]
                    csv_rows.append(
                        {
                            "team": team_label,
                            "lane": lane,
                            "player": name,
                            "tier": player_map[name]["tier"],
                            "score": player_map[name]["score"],
                        }
                    )
            csv_data = pd.DataFrame(csv_rows).to_csv(index=False).encode("utf-8-sig")
            st.download_button(
                "결과 CSV 다운로드",
                data=csv_data,
                file_name="lol_custom_match_result.csv",
                mime="text/csv",
                use_container_width=True,
            )
    else:
        st.info("플레이어와 조건을 입력한 뒤 '팀 생성'을 누르세요.")

st.divider()
st.markdown(
    """
### 실행 방법
1. 위 코드를 `app.py`로 저장
2. 터미널에서 아래 실행
```bash
pip install streamlit pandas
streamlit run app.py
```

### 현재 버전 특징
- 정확히 10명 기준 5:5 팀 생성
- 티어 점수 반영
- 가능한 라인 반영
- 같은 팀으로 묶기 / 반대 팀으로 분리하기 반영
- 점수 차이가 가장 작은 조합 탐색

### 다음 단계로 확장 가능한 것
- 주라인 / 부라인 가중치
- 같은 티어라도 LP 세분화
- 특정 두 사람은 같은 라인 금지
- 여러 후보 조합 Top 5 추천
- 밸런스 점수 공식 커스텀
"""
)
