#File Classification System
file-classifier
CLI tool that scans, classifies, and reports on file systems by type, size, and structure.

# =========================================
# CONFIGURATION
# =========================================

SMALL_FILE_THRESHOLD = 50
DEEP_ANALYSIS_LIMIT = 200


# =========================================
# INPUT LAYER
# =========================================

def get_number_of_files():
    while True:
        try:
            n = int(input("أدخل عدد الملفات: "))
            if n <= 0:
                raise ValueError("العدد يجب أن يكون أكبر من صفر.")
            return n
        except ValueError as e:
            print(f"خطأ: {e}")


def get_file_data(index):
    while True:
        try:
            print(f"\n--- الملف رقم {index+1} ---")
            name = input("اسم الملف: ").strip()
            if not name:
                raise ValueError("الاسم لا يمكن أن يكون فارغًا.")

            size = int(input("الحجم: "))
            if size < 0:
                raise ValueError("الحجم لا يمكن أن يكون سالبًا.")

            file_type = int(input("النوع (1=نص، 2=بايناري): "))
            if file_type not in (1, 2):
                raise ValueError("النوع غير صالح.")

            return {
                "name": name,
                "size": size,
                "type": "Text" if file_type == 1 else "Binary",
                "status": "Pending",
                "error": None
            }

        except ValueError as e:
            print(f"خطأ: {e}")


# =========================================
# PROCESSING LAYER
# =========================================

def process_file(file):
    size = file["size"]

    if size <= SMALL_FILE_THRESHOLD:
        file["status"] = "Success"
        return

    try:
        if size > DEEP_ANALYSIS_LIMIT:
            raise RuntimeError("فشل في المعالجة العميقة (حجم كبير جداً).")

        file["status"] = "Success"

    except Exception as e:
        file["status"] = "Failed"
        file["error"] = str(e)


# =========================================
# STATISTICS LAYER
# =========================================

def calculate_statistics(files):
    total_size = sum(f["size"] for f in files)

    return {
        "total_files": len(files),
        "small_files": sum(1 for f in files if f["size"] <= SMALL_FILE_THRESHOLD),
        "large_files": sum(1 for f in files if f["size"] > SMALL_FILE_THRESHOLD),
        "success_files": sum(1 for f in files if f["status"] == "Success"),
        "failed_files": sum(1 for f in files if f["status"] == "Failed"),
        "text_files": sum(1 for f in files if f["type"] == "Text"),
        "binary_files": sum(1 for f in files if f["type"] == "Binary"),
        "max_size": max(f["size"] for f in files),
        "min_size": min(f["size"] for f in files),
        "average_size": total_size / len(files)
    }


# =========================================
# REPORT LAYER
# =========================================

def generate_report(files, stats):
    print("\n=========== التقرير النهائي ===========")
    for key, value in stats.items():
        print(f"{key}: {value}")

    print("\n--- الملفات الفاشلة ---")
    failed = [f for f in files if f["status"] == "Failed"]

    if not failed:
        print("لا توجد ملفات فاشلة.")
    else:
        for f in failed:
            print(f"{f['name']} -> {f['error']}")


# =========================================
# MAIN ENTRY POINT
# =========================================

def main():
    files = []
    n = get_number_of_files()

    for i in range(n):
        files.append(get_file_data(i))

    for f in files:
        process_file(f)

    stats = calculate_statistics(files)
    generate_report(files, stats)


if __name__ == "__main__":
    main()
