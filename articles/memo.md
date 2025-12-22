コンテナのUID,GID バインドマウント


コンテナのログについて調査
MONTH="2020-08"
START="${MONTH}-01T00:00:00"
END="$(date -d "${MONTH}-01 +1 month" +%Y-%m-01)T00:00:00"

BASE="/var/log/jupyterhub-xxxxxx"
OUTDIR="${BASE}/exports"
mkdir -p "$OUTDIR"

cat ${BASE}/dockerlogs/jupyterhub-offline_*.log 2>/dev/null \
| awk -v start="$START" -v end="$END" '
  {
    ts=$1
    sub(/\.[0-9]+Z$/, "Z", ts)
    if (ts >= start && ts < end) print
  }
' > "${OUTDIR}/dockerlogs_full_${MONTH}.log"

grep -Ei "Server .* is ready" "${OUTDIR}/dockerlogs_full_${MONTH}.log" \
  > "${OUTDIR}/dockerlogs_ready_${MONTH}.log"





どうやって “古い” を判別してる？
-mtime +90 の意味

mtime = 最終更新日時（最後に内容が書き換わった時刻）

-mtime +90 = “最終更新が90日より前” のファイルを対象にする

つまり、

今日が 2025-12-22 だとしたら

ざっくり 2025-09-23 より前に最後に更新された .log が「古い」と判定されるイメージ。

※ find の日数は「24時間単位」で数えるので、境界はきっちり日付ジャストではなく“約”になることがある。

find "${LOG_BASE}" \
  -type f \
  -name "*.log" \
  -mtime +${LOG_RETENTION_DAYS} \
  -delete \
  2>/dev/null || true

find "${LOG_BASE}"
→ ./logs 配下を探す

-type f
→ ファイルだけ（ディレクトリは除外）

-name "*.log"
→ 拡張子 .log だけ

-mtime +90
→ 最終更新が90日より前

-delete
→ 見つかったものを削除

2>/dev/null
→ エラー表示を捨てる（権限エラー等）

|| true
→ 何か失敗してもスクリプト全体は止めない





STS方式で、OSユーザをセッション情報に刻印できるか

