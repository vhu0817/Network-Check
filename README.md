# Network-Check

# open "x-apple.systempreferences:com.apple.preference.security?Privacy_AllFiles"

# sudo python3 - << 'EOF'
import plistlib, datetime

path = "/Library/Preferences/com.apple.wifi.known-networks.plist"
with open(path, "rb") as f:
    data = plistlib.load(f)

def extract_dates(obj):
    dates = []
    if isinstance(obj, datetime.datetime):
        dates.append(obj)
    elif isinstance(obj, dict):
        for val in obj.values():
            dates.extend(extract_dates(val))
    elif isinstance(obj, list):
        for item in obj:
            dates.extend(extract_dates(item))
    return dates

rows = []
for k, v in data.items():
    if not isinstance(v, dict):
        continue
    ssid = v.get("SSID_STR") or v.get("SSID") or k
    if isinstance(ssid, bytes):
        ssid = ssid.decode("utf-8", errors="replace")
    elif isinstance(ssid, str) and ssid.startswith("wifi.network.ssid."):
        ssid = ssid.replace("wifi.network.ssid.", "", 1)

    all_dates = extract_dates(v)
    if all_dates:
        latest_ts = max(all_dates)
        local_dt = latest_ts.replace(tzinfo=datetime.timezone.utc).astimezone()
        rows.append((local_dt, str(ssid)))

rows.sort(key=lambda x: x[0], reverse=True)
h1, h2 = "Latest Timestamp (IST)", "Wi-Fi Network Name (SSID)"
print(f"{h1:<26} | {h2}")
print("-" * 65)
for dt, ssid in rows:
    t_str = dt.strftime("%Y-%m-%d %H:%M:%S %z")
    print(f"{t_str:<26} | {ssid}")
EOF
