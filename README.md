#Log Analysis System
log-analysis-system
CLI tool for parsing multi-format log files using regex, detecting anomalies and generating statistical reports.

import re
from collections import defaultdict, Counter
from dataclasses import dataclass
from typing import List, Dict


# =========================================
# CONFIGURATION
# =========================================

LOG_PATTERN = re.compile(
    r"(?P<date>\d{4}-\d{2}-\d{2})\s+"
    r"(?P<time>\d{2}:\d{2}:\d{2})\s+"
    r"\[(?P<level>INFO|WARNING|ERROR|DEBUG)\]\s+"
    r"(?P<message>.*)"
)


# =========================================
# DATA MODEL
# =========================================

@dataclass
class LogEntry:
    timestamp: str
    level: str
    message: str


# =========================================
# PARSER LAYER
# =========================================

def parse_log_line(line: str):
    match = LOG_PATTERN.match(line.strip())
    if not match:
        return None

    return LogEntry(
        timestamp=f"{match.group('date')} {match.group('time')}",
        level=match.group("level"),
        message=match.group("message")
    )


def load_logs(file_path: str) -> List[LogEntry]:
    entries = []

    with open(file_path, "r", encoding="utf-8") as f:
        for line in f:
            entry = parse_log_line(line)
            if entry:
                entries.append(entry)

    return entries


# =========================================
# ANALYSIS LAYER
# =========================================

def compute_stats(entries: List[LogEntry]) -> Dict:
    stats = Counter(entry.level for entry in entries)

    total = len(entries)

    return {
        "total_logs": total,
        "info": stats.get("INFO", 0),
        "warning": stats.get("WARNING", 0),
        "error": stats.get("ERROR", 0),
        "debug": stats.get("DEBUG", 0),
    }


# =========================================
# PATTERN DETECTION ENGINE
# =========================================

def detect_error_burst(entries: List[LogEntry], window_size=5):
    """
    Detect consecutive error spikes.
    """
    bursts = []
    window = []

    for i, entry in enumerate(entries):
        window.append(entry.level)

        if len(window) > window_size:
            window.pop(0)

        if window.count("ERROR") >= 3:
            bursts.append(i)

    return bursts


def detect_most_common_errors(entries: List[LogEntry]):
    errors = [e.message for e in entries if e.level == "ERROR"]
    return Counter(errors).most_common(3)


def detect_suspicious_patterns(entries: List[LogEntry]):
    """
    Simple heuristic security detection:
    - repeated failures
    - frequent errors in short span
    """

    patterns = {
        "error_bursts": detect_error_burst(entries),
        "top_errors": detect_most_common_errors(entries),
    }

    return patterns


# =========================================
# REPORT LAYER
# =========================================

def generate_report(entries: List[LogEntry]):
    stats = compute_stats(entries)
    patterns = detect_suspicious_patterns(entries)

    report = {
        "statistics": stats,
        "patterns": patterns
    }

    return report


def print_report(report: Dict):
    print("\n========== LOG ANALYSIS REPORT ==========\n")

    print("📊 STATISTICS")
    for k, v in report["statistics"].items():
        print(f"{k}: {v}")

    print("\n🧠 PATTERN DETECTION")

    print("\n- Error bursts detected at indexes:")
    print(report["patterns"]["error_bursts"])

    print("\n- Most common errors:")
    for msg, count in report["patterns"]["top_errors"]:
        print(f"{msg} -> {count} times")


# =========================================
# MAIN PIPELINE
# =========================================

def main(file_path: str):
    entries = load_logs(file_path)

    if not entries:
        print("No valid logs found")
        return

    report = generate_report(entries)
    print_report(report)


if __name__ == "__main__":
    main("logs.txt")