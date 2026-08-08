#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
강서/강남/강북 사업소 시장모니터링 대시보드 - 다가오는 일정(3일 이내) 텔레그램 알림
매일 GitHub Actions에서 자동 실행됨 (.github/workflows/telegram-alert.yml 참고)
"""
import os
import sys
import json
import urllib.request
from datetime import date, timedelta

FIREBASE_DATA_URL = "https://add-project-74efc-default-rtdb.asia-southeast1.firebasedatabase.app/data.json"
DAYS_AHEAD = 3  # 며칠 전부터 알림 보낼지

BOT_TOKEN = os.environ.get("TELEGRAM_BOT_TOKEN", "")
CHAT_ID = os.environ.get("TELEGRAM_CHAT_ID", "")


def fetch_json(url):
    with urllib.request.urlopen(url, timeout=20) as res:
        return json.loads(res.read().decode("utf-8"))


def collect_upcoming_events(data, days_ahead):
    today = date.today()
    end_day = today + timedelta(days=days_ahead)
    offices = (data or {}).get("offices", {}) or {}

    upcoming = []
    for office_name, sites in offices.items():
        if not isinstance(sites, list):
            continue
        for site in sites:
            if not isinstance(site, dict):
                continue
            site_name = site.get("name", "(이름없음)")
            events = site.get("events", [])
            if not isinstance(events, list):
                continue
            for ev in events:
                if not isinstance(ev, dict):
                    continue
                ev_date_str = ev.get("date", "")
                label = ev.get("label", "")
                if not ev_date_str or not label:
                    continue
                try:
                    y, m, d = [int(x) for x in ev_date_str.split("-")]
                    ev_date = date(y, m, d)
                except Exception:
                    continue
                if today <= ev_date <= end_day:
                    upcoming.append((ev_date, office_name, site_name, label))

    upcoming.sort(key=lambda x: x[0])
    return upcoming


def build_message(upcoming, days_ahead):
    if not upcoming:
        return None  # 알릴 일정 없으면 메시지 자체를 안 보냄

    today = date.today()
    lines = [f"📅 {days_ahead}일 이내 다가오는 일정 ({today.strftime('%Y.%m.%d')} 기준)", ""]
    for ev_date, office_name, site_name, label in upcoming:
        d_day = (ev_date - today).days
        d_label = "오늘" if d_day == 0 else f"D-{d_day}"
        lines.append(f"[{d_label}] {ev_date.strftime('%m.%d')} · {office_name} · {site_name}")
        lines.append(f"   {label}")
    return "\n".join(lines)


def send_telegram(message):
    if not BOT_TOKEN or not CHAT_ID:
        print("TELEGRAM_BOT_TOKEN / TELEGRAM_CHAT_ID 가 설정되지 않았습니다.", file=sys.stderr)
        sys.exit(1)
    url = f"https://api.telegram.org/bot{BOT_TOKEN}/sendMessage"
    payload = json.dumps({"chat_id": CHAT_ID, "text": message}).encode("utf-8")
    req = urllib.request.Request(url, data=payload, headers={"Content-Type": "application/json"})
    with urllib.request.urlopen(req, timeout=20) as res:
        print(res.read().decode("utf-8"))


def main():
    data = fetch_json(FIREBASE_DATA_URL)
    upcoming = collect_upcoming_events(data, DAYS_AHEAD)
    message = build_message(upcoming, DAYS_AHEAD)
    if message is None:
        print("알림 보낼 일정 없음, 조용히 종료합니다.")
        return
    send_telegram(message)
    print("텔레그램 알림 전송 완료.")


if __name__ == "__main__":
    main()
