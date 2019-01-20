```
apt-get install sphinxsearch
mkdir /etc/sphinxsearch/dicts
wget http://sphinxsearch.com/files/dicts/en.pak -O /etc/sphinxsearch/dicts/en.pak
wget http://sphinxsearch.com/files/dicts/ru.pak -O /etc/sphinxsearch/dicts/ru.pak 
```

```
# nano /etc/sphinxsearch/sphinx.conf

common {
    lemmatizer_base = /etc/sphinxsearch/dicts
}

source mysql {
    type = mysql
    sql_host = localhost
    sql_user = anilibria
    sql_pass = anilibria
    sql_db = anilibria

    # Sphinx return empty result on cyrillic query
    # http://sphinxsearch.com/forum/view.html?id=11176
    sql_query_pre = SET NAMES utf8

    sql_query_range = select min(id), max(id) from `xrelease`
    sql_range_step = 2048

    sql_query = select id, name, ename, genre, year, last, rating from `xrelease` where id >= $start and id <= $end
    
    # http://sphinxsearch.com/forum/view.html?id=15403
    sql_attr_uint = last
    sql_attr_uint = rating
}

index anilibria {
    source = mysql
    
    # ё => е, э => е
    # http://sphinxsearch.com/docs/current/conf-charset-table.html
    # http://sphinxsearch.com/forum/view.html?id=7280 
    charset_table = 0..9, A..Z->a..z, _, a..z, U+410..U+42C->U+430..U+44C, \
    U+42E..U+42F->U+44E..U+44F, U+430..U+44C, U+44E..U+44F, \
    U+0401->U+0435, U+0451->U+0435, U+042D->U+0435, U+044D->U+0435
    
    # Cats => Cat
    # https://habr.com/post/147745/ 
    morphology = stem_enru, soundex
 
	# https://habr.com/ru/post/147745/
    ondisk_attrs=1
    min_word_len = 3
    min_infix_len = 3
    expand_keywords = 1
    index_exact_words = 1
      
    path = /var/lib/sphinxsearch/data/anilibria
}

searchd {
    listen			= localhost:9312
    listen			= localhost:9306:mysql41
    log = /var/log/sphinxsearch/searchd.log
    query_log = /var/log/sphinxsearch/query.log
    pid_file = /var/run/sphinxsearch/searchd.pid
}
```

```
# nano /etc/cron.d/sphinxsearch

* * * * * sphinxsearch [ -x /usr/bin/indexer ] && /usr/bin/indexer --quiet --rotate --config /etc/sphinxsearch/sphinx.conf --all
```

```
/etc/init.d/sphinxsearch start
```
