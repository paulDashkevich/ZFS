#zfs
otus
I. определить алгоритм с лучшим сжатием:

        1. Натягиваем ZFS на диск sdb: mkfs.zfs /dev/sdb
        2. Создаём пул: zpool create test /dev/sdb
        3. Смотрим на результат:
```
                zfs list
                NAME   USED  AVAIL     REFER  MOUNTPOINT
                test    93K  1.75G       24K  /test
```
        4. Создаём датасеты по названию методов сжатия (для удобства)

```
                                zfs create test/lz4
                                zfs create test/gzip
                                zfs create test/lzjb
                                zfs create test/zle

                                **zfs list**
                NAME        USED  AVAIL     REFER  MOUNTPOINT
                test        212K  1.75G       29K  /test
                test/gzip    24K  1.75G       24K  /test/gzip
                test/lz4     24K  1.75G       24K  /test/lz4
                test/lzjb    24K  1.75G       24K  /test/lzjb
                test/zle     24K  1.75G       24K  /test/zle
```

        5. Устанавливаем на каждый датасет метод компрессии:

```
                                zfs set compression=lz4 test/lz4
                                zfs set compression=gzip-9 test/gzip
                                zfs set compression=lzjb test/lzjb
                                zfs set compression=zle test/zle
                проверяем наличие заданной компрессии zfs get compression
```
                **zfs get compression**
                NAME       PROPERTY     VALUE     SOURCE
                test       compression  off       default
                test/gzip  compression  gzip-9    local
                test/lz4   compression  lz4       local
                test/lzjb  compression  lzjb      local
                test/zle   compression  zle       local
```


        6. До начала тестов со сжатием фиксируем пустой пул командой **zpool list**
```
                zpool list
                NAME   SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
                test  1.88G   276K  1.87G        -         -     0%     0%  1.00x    ONLINE  -
```

        7. Качаем файл из задания и копируем во все датасеты

```
        wget -O War_and_Peace.txt http://www.gutenberg.org/ebooks/2600.txt.utf-8
                cp /home/paul/zfs/War_and_Peace.txt -R /test/gzip
                cp /home/paul/zfs/War_and_Peace.txt -R /test/lzjb
                cp /home/paul/zfs/War_and_Peace.txt -R /test/zle
                cp /home/paul/zfs/War_and_Peace.txt -R /test/lz4
```
        8. Степень сжатия можно узнать с помощью свойства **compressratio**
```
                **zfs get compressratio**
                NAME       PROPERTY       VALUE  SOURCE
                test       compressratio  1.06x  -
                test/gzip  compressratio  1.08x  -
                test/lz4   compressratio  1.08x  -
                test/lzjb  compressratio  1.07x  -
                test/zle   compressratio  1.08x  -
                Реальный размер файла
                ls -lh War_and_Peace.txt
-rw-r--r--. 1 root root 1.2M May  7  2016 War_and_Peace.txt
```

**II. Определить настройки pool’a**

        1. Загружаем архив
 wget --no-check-certificate 'https://docs.google.com/uc?export=download&id=1KRBNW33QWqbvbVHa3hLJivOAt60yukkg' -O zfs_task1.tar.gz
        2. Распаковываем:
                tar xvzf zfs_task1.tar.gz

        3. Собираем зеркало из vdev
                zpool create storage/otus /home/paul/zfs/zpoolexport/filea /home/paul/zfs/zpoo                      lexport/fileb
        4. Смотрим, что имеем:
```
                 **zfs list**
                NAME           USED  AVAIL     REFER  MOUNTPOINT
                otus            96K   832M       24K  /otus
                test          6.27M  1.74G     1.28M  /test
                test/gzip     1.18M  1.74G     1.18M  /test/gzip
                test/lz4      1.18M  1.74G     1.18M  /test/lz4
                test/lzjb     1.19M  1.74G     1.19M  /test/lzjb
                test/storage    24K  1.74G       24K  /test/storage
                test/zle      1.18M  1.74G     1.18M  /test/zle
```
        5. Командами zfs определить настройки:
                5.1 размер хранилища zfs list otus
```
                NAME   USED  AVAIL     REFER  MOUNTPOINT
                otus   105K   832M       24K  /otus
```
                5.2 тип pool
```
                **zpool status otus**
                pool: otus
                state: ONLINE
                scan: none requested
                config:

                NAME                               STATE     READ WRITE CKSUM
                otus                              ONLINE       0     0     0
                /home/paul/zfs/zpoolexport/filea  ONLINE       0     0     0
                /home/paul/zfs/zpoolexport/fileb  ONLINE       0     0     0
```
                5.3 значение recordsize
```
                **zfs get recordsize**
                NAME          PROPERTY    VALUE    SOURCE
                otus          recordsize  128K     default
                test          recordsize  128K     default
                test/gzip     recordsize  128K     default
                test/lz4      recordsize  128K     default
                test/lzjb     recordsize  128K     default
                test/storage  recordsize  128K     default
                test/zle      recordsize  128K     default
```
                5.4 какое сжатие используется
```
                **zfs get compress**
                NAME          PROPERTY     VALUE     SOURCE
                otus          compression  off       default
                test          compression  off       default
                test/gzip     compression  gzip-9    local
                test/lz4      compression  lz4       local
                test/lzjb     compression  lzjb      local
	        test/storage  compression  off       default
                test/zle      compression  zle       local
```
                5.5 какая контрольная сумма используется
```
                **zpool get checksum**
                NAME  PROPERTY  VALUE      SOURCE
                otus  checksum  on         default
```
**III Найти сообщение от преподавателей**

        1. Скачать из гугл-диска дамп датасета с инфо:
```
         wget --no-check-certificate 'https://docs.google.com/uc?export=download&id=1gH8gCL9y7Nd5Ti3IRmplZPF1XjzxeRAG' -O otus_task2.file

```
        2. Заливаем содержимое в пул otus:
```
        zfs recv  otus/task2 < /otus/otus_task2.file
```
        3. В папке file_mess смотрим файлы ls -lah и находим secret_message
```
        https://github.com/sindresorhus/awesome
```

