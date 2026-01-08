
### `grep` - поиск текста в файлах
```bash
grep "текст" file.txt             # найти текст в файле
grep -r "error" /var/log/         # рекурсивный поиск
grep -i "warning" file.txt        # без учета регистра
grep -v "test" file.txt           # исключить строки с "test"
grep -n "pattern" file.txt        # показать номера строк
```

##  **Полезные комбинации**

### Мониторинг и поиск вместе:
```bash
# Найти файлы с ошибками в логах
find /var/log -name "*.log" -exec grep -l "error" {} \;

# Найти большие файлы и отсортировать
find / -size +500M -exec ls -lh {} \; 2>/dev/null

ls -l | grep rc.
```

