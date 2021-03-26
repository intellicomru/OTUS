### Курсовой проект ### 
Данный проект ориентирован на использование технологий изученных на курсе PostgreSQL OTUS  
Установка и настройка PostgreSQL  
Установка и использование PostgreSQL и Kubernetes  
Работа с большим объемом реальных данных  
Виды индексов. Работа с индексами и оптимизация запросов  
Различные виды join'ов. Применение и оптимизация  
Секционирование  
Полнотекстовый индекс.  


 1. Обновим Debian  
apt-get -y update  
apt-get -y upgrade  


apt-get -y install nginx   
systemctl start nginx   
systemctl enable nginx   

apt-get -y install apache2   
sed -i "s/Listen 80/Listen 127.0.0.1:8080/" /etc/apache2/ports.conf  

 Apache2 Real IP  
 ```
vi /etc/apache2/mods-available/remoteip.conf  
<IfModule remoteip_module>
  RemoteIPHeader X-Forwarded-For
  RemoteIPTrustedProxy 127.0.0.1/8
</IfModule>
```

Активируем модуль:  

a2enmod remoteip  

cgi   

vi /etc/apache2/sites-available/000-default.conf  


```
 <VirtualHost *:8080>
ServerName myhost

ServerAdmin webmaster@localhost
DocumentRoot /var/www/html

ErrorLog ${APACHE_LOG_DIR}/error.log
CustomLog ${APACHE_LOG_DIR}/access.log combined

ScriptAlias /cgi-bin/ /var/www/cgi-bin/
<Directory «/var/www/cgi-bin/»>
     AllowOverride None
     Options +ExecCGI -MultiViews +SymLinksIfOwnerMatch
     Require all granted
</Directory>

</VirtualHost>
```

a2enmod cgi  
mkdir -p /var/www/cgi-bin/  
systemctl restart apache2    

  конфигурим  nginx   

systemctl restart nginx   

 поставили гит   

cd  /var/www/    
apt-get install git   
git init   
git remote add origin https://github.com/intellicomru/project-otus.git   
git pull origin master    

###### ставим постгрю и модули Perl   ######  

sudo apt-get -y install postgresql    
 pg_lsclusters  
 
 ```
Ver Cluster Port Status Owner    Data directory              Log file
11  main    5432 online postgres /var/lib/postgresql/11/main /var/log/postgresql/postgresql-11-main.log
```

sudo -u postgres psql  
CREATE ROLE otus LOGIN PASSWORD '1234567890';  
CREATE DATABASE otus;  
 ALTER DATABASE otus OWNER TO otus;  
 \q  
 
 --После этого нужно добавить для этого пользователя строчку в файл   
 /etc/postgresql/11/main/pg_hba.conf   
 local   all             otus                             md5 
 
sudo systemctl stop postgresql@11-main     
sudo systemctl start postgresql@11-main  



 **perl вспомогательные модули. просто для удобства**  
   
```
sudo -s 
apt-get install build-essential
cpan 
install CGI 
install lib::abs
install uni::perl
install DBI
install YAML::XS
install Spreadsheet::Read
install LWP::UserAgent
install JSON

```

#### драйвера для связи перла и базы #### 
sudo apt-get install libpq-dev
apt-get install libdbd-pg-perl


создаем таблицы

```
psql -U otus -d otus
create schema muzik;

create table muzik.file_data(
	id bigserial NOT NULL,
performers varchar ,
name_orig varchar,
album_name varchar,
author_music varchar, 
author_text varchar,
publisher varchar,
duration varchar,
public_year varchar,
genre varchar,
filename varchar,
link varchar,
size int default 0,
md5 varchar,
isrc varchar,
icpn varchar,
CONSTRAINT catalog_pkey PRIMARY KEY (id)
);
```
заполняем данными : 

~~~
cat /home/alex/mp3_data_all.csv | psql -h 127.0.0.1 -p 5432 -U otus -d otus  -c "COPY muzik.file_data (performers ,name_orig ,album_name ,author_music ,author_text ,publisher,duration ,public_year ,genre ,filename ,link ,size,md5,isrc,icpn) FROM STDIN DELIMITER '~'   quote E'\b' escape '\"' CSV" 

~~~

установка и настройка кластера тут : [тут](https://github.com/intellicomru/OTUS/blob/main/prj-okrujenie.md).

##### подключаемся к кластеру через промежуточную машину пока  #####
psql -h 10.154.0.6  -U postgres  

CREATE ROLE otus LOGIN PASSWORD '1234567890';  
CREATE DATABASE otus;  
 ALTER DATABASE otus OWNER TO otus;  
 \q  

###### добавляем в кластер доступ для otus на управляющей машине  ###### 

локально фовардим kubectl port-forward --namespace default svc/pgsql-ha-postgresql-ha-pgpool 8888:5432

**строчка для записи доступа в куб.**     

 kubectl exec -it $(kubectl get pods -l app.kubernetes.io/component=pgpool,app.kubernetes.io/name=postgresql-ha -o jsonpath='{.items[0].metadata.name}') -- pg_md5 -m --config-file="/opt/bitnami/pgpool/conf/pgpool.conf" -u "otus" "1234567890"   

заливаем данные в кластер   

cat /home/alex/mp3_data_all.csv | psql -h 10.154.0.6 -p 5432 -U otus -d otus  -c "COPY muzik.file_data (performers ,name_orig ,album_name ,author_music ,author_text ,publisher,duration ,public_year ,genre ,filename ,link ,size,md5,isrc,icpn) FROM STDIN DELIMITER '~'  CSV"   



### Строим индексы  ###
Реализовать индекс для полнотекстового поиска  
добавляем колонку tsvector для полнотекстового поиска   

**ИCПОЛНИТЕЛЬ**     
alter table muzik.file_data add column ft_performers tsvector;    
заполняем данными :    
update muzik.file_data set ft_performers=to_tsvector(performers);    
CREATE INDEX muzik_file_data_performers ON muzik.file_data USING GIN (ft_performers);    

**НАЗВАНИЕ**    
alter table muzik.file_data add column ft_name_orig tsvector;     
заполняем данными :     
update muzik.file_data set ft_name_orig=to_tsvector(name_orig);    
CREATE INDEX muzik_file_data_ft_name_orig ON muzik.file_data USING GIN (ft_name_orig);   

**ISRC**
CREATE INDEX muzik_file_data_lower_isrc ON muzik.file_data (lower(isrc));  

**ПО всем полям**  
alter table muzik.file_data add column ft_all tsvector;     
заполняем данными :     
update muzik.file_data set ft_all=to_tsvector(concat(performers,' ',name_orig,' ',album_name,' ',author_music,' ',author_text,' ',publisher,' ',genre,' ',isrc,' ',icpn));    

CREATE INDEX muzik_file_data_ft_all ON muzik.file_data USING GIN (ft_all);   


performers varchar ,
name_orig varchar,
album_name varchar,
author_music varchar, 
author_text varchar,
publisher varchar,
duration varchar,
public_year varchar,
genre varchar,
filename varchar,
link varchar,
size int default 0,
md5 varchar,
isrc varchar,
icpn varchar,

### Секционирование ###
**Цель :**     

 на входе поступают списки произведений в формате двух колонок   

 Исполнитель ; Название произведения   

необходимо найти и сопоставить их с данными контента  

пример входящего файла :   
````
artist;	title
DJ DimixeR feat. Max Vertigo;	Sambala (Wallmers Remix)
DJ DimixeR feat. Max Vertigo;	Sambala
DJ DimixeR feat. Max Vertigo;	Sambala (Club Mix)
DJ DimixeR feat. Max Vertigo;	Sambala (K 11 remix)
DJ DimixeR feat. Max Vertigo;	Sambala (Jimmy Jaam Radio Remix)
DJ DimixeR feat. Max Vertigo;	Sambala (Jimmy Jaam Remix)
DJ DimixeR feat. Max Vertigo;	Sambala (Menshee Radio Edit)
DJ DimixeR feat. Max Vertigo;	Sambala (Menshee Remix)
````

Решение :   

Создаем кеширующую партицированную таблицу :   

create table muzik.performer_title (
 id int,
 performer varchar,
 title varchar,
 isrc varchar
 ) partition by hash(performer,title);


создаем партиции   

DO
$$
declare
    i   int;
begin
FOR i IN 1..100 LOOP
   execute format('create table muzik.performer_title_%s partition of muzik.performer_title FOR VALUES WITH (MODULUS 100, REMAINDER %s)', i, i-1);
END loop;
end;
$$;



**Создаем функцию нормализации**  

CREATE OR REPLACE FUNCTION public.normalize_title(t text)
 RETURNS text
 LANGUAGE plpgsql
AS $function$
 DECLARE
  res    text;
 BEGIN
res=lower(t);
res=regexp_replace(res, '[\(\)\s\+\-\.\,\"\:\;\\!\§\@\#\$\%\^\&\*\/\\\''\~\`\>\<\[\]]', ' ', 'g');
res=regexp_replace(res, '\s+', ' ', 'g');
res=regexp_replace(res, '^\s+|\s+$', '', 'g');
return res;
END
$function$
;


**Создаем триггер чтобы заполнять ее новыми данными после инсерта в основную таблицу** 

CREATE OR REPLACE FUNCTION public._file_data_after_insert()
 RETURNS trigger
 LANGUAGE plpgsql
AS $function$
BEGIN
   insert into muzik.performer_title (hgc_id,performer,title,isrc) values(NEW.id,public.normalize_title(NEW.performers),public.normalize_title(NEW.name_orig),NEW.isrc);
    RETURN NEW.id;
END;
$function$
;
create trigger add_perormer_title_after_insert after
insert
    on
    muzik.file_data for each row execute procedure public._file_data_after_insert();


заполняем старыми данными   

insert into  muzik.performer_title (id,performer,title,isrc)
select id,public.normalize_title(performers),public.normalize_title(name_orig),isrc from muzik.file_data;


Пример использования.   
сначала мы нормализуем, чтобы хеш работал однозначно. 

SELECT public.normalize_title('Antti Ketonen') performer,public.normalize_title('Olisitpa sylissäni') title ;

select pt.id,pt.performer,pt.title,pt.isrc
from muzik.performer_title pt
 where pt.performer ='antti ketonen' and pt.title = 'olisitpa sylissäni';
                hgc_id                |   performer   |       title        |     isrc    
--------------------------------------+---------------+--------------------+--------------
 5f527f1b-0882-6400-0000-12f8fade1e64 | antti ketonen | olisitpa sylissäni | FIWMA1700103
 5ed6db9b-06d6-be00-0000-c78a9555e0ec | antti ketonen | olisitpa sylissäni | FIWMA1700103
 5f528dd9-007e-d900-0000-ecd77ab8c8b1 | antti ketonen | olisitpa sylissäni | FIWMA1700103
 5f528241-0067-9000-0000-393527063fb1 | antti ketonen | olisitpa sylissäni | FIWMA1700103
 5de7e105-0000-0000-0000-0000dc1c855a | antti ketonen | olisitpa sylissäni | FIWMA1700103
 5de7da8d-0000-0000-0000-0000d0627638 | antti ketonen | olisitpa sylissäni | FIWMA1700103
 5de66bbb-0000-0000-0000-000007d472e6 | antti ketonen | olisitpa sylissäni | FIWMA1700103
 5ed6dbed-0013-6000-0000-3f19b5c604cd | antti ketonen | olisitpa sylissäni | FIWMA1700103
 5f525775-0439-1900-0000-d1f1404642a9 | antti ketonen | olisitpa sylissäni | FIWMA1700103
 5de7953b-0000-0000-0000-0000e69b44f6 | antti ketonen | olisitpa sylissäni | FIWMA1700103
(10 rows)

### дока ###
счетчики как ускорить 
 https://habr.com/ru/post/276055/   

SELECT reltuples::bigint
FROM pg_catalog.pg_class
WHERE relname = 'muzik.file_data';


