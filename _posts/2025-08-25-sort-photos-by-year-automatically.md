---
layout: post
---

I have a big directory that contains over 100k photos, now I want to move them into different directory by year its shoot, so EXIF information is needed, combine with powerful `find` command, it's an easy job to get it done.

As I am with FreeBSD, so the grammer and tools has a bit difference.

`pkg install exif  # install exif tool`

```
#!/bin/sh

# 指定照片目录（脚本参数或默认当前目录）
PHOTO_DIR="${1:-.}"

# 查找所有常见照片格式（使用FreeBSD兼容的语法）
find "$PHOTO_DIR" -type f \( -name "*.jpg" -o -name "*.jpeg" -o -name "*.JPG" -o -name "*.HEIC" -o -name "*.PNG" \) | while read -r file; do
  # 提取EXIF中的创建时间（DateTimeOriginal字段）
  exif_data=$(exif -t "DateTimeOriginal" -m "$file" 2>/dev/null)
  
  # 若无法提取EXIF，跳过该文件
  if [ -z "$exif_data" ]; then
    echo "跳过（无EXIF数据）: $file"
    continue
  fi

  # 提取年份（格式：YYYY:MM:DD HH:MM:SS → 取前4位）
  year=$(echo "$exif_data" | cut -d: -f1)

  # 验证年份是否合法（4位数字）
  if echo "$year" | grep -qE '^[0-9]{4}$'; then
    # 创建目标目录（如果不存在）
    target_dir="$PHOTO_DIR/$year"
    mkdir -p "$target_dir"

    # 移动文件到目标目录
    mv -v "$file" "$target_dir/"
  else
    echo "无效年份: $file (EXIF: $exif_data)"
  fi
done
```

Then add excution permission

`chmod +x sort-photo.sh`

`./sort-photo.sh.sh /path/to/your/photo/`

---

As command `find` has difference between FreeBSD and Linux, hereby is the one for Linux.

```
#!/bin/sh

# 指定照片目录（脚本参数或默认当前目录）
PHOTO_DIR="${1:-.}"

# 查找所有常见照片格式（根据需要扩展）
find "$PHOTO_DIR" -type f \( \
  -iname "*.jpg" -o \
  -iname "*.jpeg" -o \
  -iname "*.tiff" -o \
  -iname "*.nef" -o \
  -iname "*.cr2" \
\) | while read -r file; do
  # 提取EXIF中的创建时间（DateTimeOriginal字段）
  exif_data=$(exif -t "DateTimeOriginal" -m "$file" 2>/dev/null)
  
  # 若无法提取EXIF，跳过该文件
  if [ -z "$exif_data" ]; then
    echo "跳过（无EXIF数据）: $file"
    continue
  fi

  # 提取年份（格式：YYYY:MM:DD HH:MM:SS → 取前4位）
  year=$(echo "$exif_data" | cut -d: -f1)

  # 验证年份是否合法（4位数字）
  if echo "$year" | grep -qE '^[0-9]{4}$'; then
    # 创建目标目录（如果不存在）
    target_dir="$PHOTO_DIR/$year"
    mkdir -p "$target_dir"

    # 移动文件到目标目录
    mv -v "$file" "$target_dir/"
  else
    echo "无效年份: $file (EXIF: $exif_data)"
  fi
done
```
