# vosk-adaptation

Адаптация большой модели русского языка http://vosk-model-ru-0.10.zip

1. Подготовка корпуса (каждое предложение должно быть с новой строки и без знаков препинания)
2. выделение уникальных слов из корпуса
    grep -oE "[А-Яа-я\\-]{3,}" corpus.txt | tr '[:lower:]' '[:upper:]' | sort | uniq > words.txt
3. построение произношений:
    1. установить программу phonetisaurus, инструкция по установке здесь: https://github.com/AdolfVonKleist/Phonetisaurus
    2. скачать g2p модель: wget https://alphacephei.com/vosk/models/ru.dic.fst
    3. форматировать исходный словарь vosk-model-ru-0.10/extra/db/ru.dict
        cat ru.dict \
          | perl -pe 's/\([0-9]+\)//;
              s/\s+/ /g; s/^\s+//;
              s/\s+$//; @_ = split (/\s+/);
              $w = shift (@_);
              $_ = $w."\t".join (" ", @_)."\n";' \
          > ru.formatted.dict
    4. построить произношения для нового словаря
        phonetisaurus-apply --model train/ru.dict.fst --word_list test.wlist -l ru.formatted.dict > words.dict
4. построение языковой модели:
    1. установить SRILM по инструкции: https://hovinh.github.io/blog/2016-04-22-install-srilm-ubuntu/
    2. построить языковую модель по той же инструкции
5. Объединение получившихся словаря и языковой модели с исходными с помощью скрипта (в качестве языковой модели использовать vosk-model-ru-0.10/extra/db/ru-small.lm): 
    python3 mergedicts.py ../extra/db/ru.dic ../extra/db/ru-small.lm words.dic lm.lm merged-words.txt merged-lm.arpa
6. компиляция нового графа HCLG.fst
7. замена старого графа новым
