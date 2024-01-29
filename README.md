# USB_info--for_Linux
Получаем информацию о подключенном USB - Flash накопителе
```python
import subprocess

def get_connected_flash_drives():
    connected_drives = []

    # Получение информации о блочных устройствах с помощью lsblk
    lsblk_output = subprocess.check_output(["lsblk", "-o", "NAME,SIZE,TYPE,MOUNTPOINT"])

    # Получение списка строк из вывода lsblk
    lsblk_lines = lsblk_output.decode().splitlines()

    for line in lsblk_lines[1:]:
        # Разделение строки на колонки
        columns = line.split()

        # Извлечение имени устройства, размера и типа
        name = columns[0]
        size = columns[1]
        drive_type = columns[2]

        # Проверка, является ли устройство съемным (TYPE="disk")
        if drive_type == "disk":
            # Используйте udevadm для получения информации о серийном номере
            try:
                udevadm_output = subprocess.check_output(["udevadm", "info", "--query=property", "--name", f"/dev/{name}"])
                serial_number = [line.split("=")[1] for line in udevadm_output.decode().splitlines() if "ID_SERIAL_SHORT" in line][0]
            except (subprocess.CalledProcessError, IndexError):
                serial_number = "N/A"

            connected_drive = {
                'device': f"/dev/{name}",
                'serial_number': serial_number,
                'size': size
            }

            connected_drives.append(connected_drive)

    return connected_drives

def main():
    drives = get_connected_flash_drives()
    if drives:
        for drive in drives:
            print(f"Устройство: {drive['device']}")
            print(f"Серийный номер: {drive['serial_number']}")
            print(f"Размер: {drive['size']}")
            print("-" * 30)
    else:
        print("Нет подключенных флеш-накопителей.")

if __name__ == "__main__":
    main()
```
