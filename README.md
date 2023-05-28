# ОреолАвтосервис
Работа с запросами в PostgreSQL. 

<details><summary>Простая выборка</summary>
1. Выдать список механиков (все столбцы таблицы mechanic).
		
```
SELECT * 
FROM mechanic
```
	
2. Выдать государственные номерные знаки, серии, номера и даты выдачи свидетельств о регистрации транспортных средств (таблица vehicle).

```
SELECT gnz, 
       ser$reg_certif, 
       num$reg_certif, 
       date$reg_certif 
FROM vehicle
```
	
3. Сформировать список автомобилей, прошедших обслуживание (таблица maintenance), путем указания их государственных номерных знаков (без дубликатов).
	
```
SELECT DISTINCT gnz  
FROM maintenance  
```
	
4. Выдать список государственных номерных знаков, серии, номера и даты выдачи свидетельств о регистрации транспортных средств в виде таблицы из двух колонок – "Государственный номерной знак", "Свидетельство о регистрации транспортного средства".
	
```
SELECT gnz "Государственный номерной знак",  
       concat_ws(', ', ser$reg_certif, 
       		       num$reg_certif, 
       		       date$reg_certif) 
       "Свидетельство о регистрации транспортного средства" 
FROM vehicle  
```
	
5. Сформировать список автомобильных заводов с указанием наименования, адреса фактического размещения и контактного телефона. Перечень должен быть отсортирован по наименованию, по алфавиту.
	
```
SELECT factory_name, 
       legal_addr,
       phone 
FROM factory 
ORDER BY factory_name
```
	
6. Составить список групп транспортных средств (таблица transpgroup) в формате <идентификатор группы>: <наименование группы> - <описание> (колонки id_tg, name и note, соответственно). Результирующий столбец должен быть именован, как "Группы транспортных средств".
	
```
SELECT concat(id_tg,': ', name,' - ', note) 
	   "Группы транспортных средств" 
FROM transpgroup
```
	
7. Составить список автомобилей с указанием их государственного номерного знака, стоимости и уплаченной суммы налога на добавленную стоимость, которая рассчитывается по ставке 20%.
	
```
SELECT gnz, 
       cost, 
       cost*0.2 "Налог" 
FROM vehicle
```
	
8. Рассчитать суммарную стоимость зарегистрированных автомобилей. Результат представить в денежном формате (длина мантиссы равна 2).
	
```
SELECT SUM(cost)::money "Суммарная стоимость" 
FROM vehicle
```
	
9. Выдать фамилии, инициалы и дату рождения механиков (получить в результирующей выборке один столбец со значениями вида "Светлов В.К., дата рождения 01.06.1967"). Для форматирования дат рождения использовать маску dd.mm.yyyy. Дать столбцу альтернативное имя "Лучшие механики предприятия". Ограничить список первыми пятью механиками, сортировку не производить.
Пример записи в ответе: Савостьянов А.В., дата рождения 23.03.1970
	
```
SELECT concat(sname_initials, ', 
       дата рождения ',to_char(born,'dd.mm.yyyy')) 
       "Лучшие механики предприятия" 
FROM mechanic 
LIMIT 5
```
	
10. Выдать фамилии, инициалы, даты рождения в формате 'dd.mm.yyyy' и возраст (в полных годах) механиков (использовать встроенную функцию age для работы с интервалами дат; выражение вида trunc((current_date-born)/365) некорректно, так как оно не учитывает високосные года).
	
```
SELECT sname_initials, 
       to_char(born,'dd.mm.yyyy') born, 
       age(born)  
FROM mechanic
```
	
11. Рассчитать отношение стоимости каждого автомобиля к его пробегу в километрах с точностью до копейки. Результат представить в виде: "<государственный номерной знак>-<значение отношения стоимости к пробегу> руб/км".

```
SELECT concat(gnz, ' - ',cost::money/run, 'руб/км') 
FROM vehicle
```
	
12. Сформировать список автомобилей (государственный номерной знак) с указанием в отдельном столбце даты, а в отдельном столбце времени прохождения обслуживания (таблица maintenance).

```
SELECT gnz, 
       to_char(date_work, 'dd.mm.yyyy')  "Date", 
       to_char(date_work, 'hh24:mi:ss')  "Time" 
FROM maintenance
```
	
13. Сформировать ведомость амортизационной стоимости автомобилей, учитывая, что за каждый полный год объект учета теряет 7% первоначальной стоимости. Для автомобилей, амортизационная стоимость которых меньше нуля, указывать отрицательную величину. Возраст автомобиля считать от даты ввода в эксплуатацию – date_use (таблица vehicle). В ведомость включить государственные номерные знаки, дату ввода в эксплуатацию и остаточную стоимость.
	
```
SELECT gnz, 
       date_use, 
       cost-(cost*0.07*(EXTRACT(YEAR FROM age(date_use)))) 
       "Амортизационная стоимость" 
FROM vehicle
```
	
14. Сформировать список автомобилей с указанием дня недели и порядкового дня года, в который они выпущены (столбец date_made таблицы vehicle). Результат оформить в виде одного столбца с именем "День недели и день года выпуска".

```
SELECT concat(to_char(date_made, 'Day'), ' ', 
       date_part('doy', date_made))  
       "День недели и день выпуска" 
FROM vehicle
```
</details>
	
<details>
<summary>Отбор по условию и сортировка (по одной таблице)</summary>

15. Найти российские автомобильные заводы, у которых почтовый и фактический адреса совпадают. Сформировать список с именами, фактическими адресами и контактными телефонами предприятий. 

```
SELECT factory_name, 
       legal_addr, 
       phone 
FROM factory 
WHERE phone LIKE '+7%' 
      AND 
      post_addr = legal_addr
```
	
16. Составить список механиков, имеющих трудовой стаж (столбец certif_date) более 13 лет. Выдать фамилии и инициалы механиков, даты выдачи сертификатов и приема на работу, трудовой стаж (полных лет), отсортировать список по возрастанию трудового стажа. 
	
```
SELECT sname_initials, 
       certif_date, 
       work_in_date, 
       EXTRACT(YEAR FROM age(certif_date)) "Стаж" 
FROM mechanic 
WHERE EXTRACT(YEAR FROM age(certif_date))>=13 
ORDER BY 4 ASC
```

17. Найти автомобили, для которых НДС, уплаченный при приобретении, превосходит 600 000 рублей (НДС рассчитывается по ставке 20% от суммы платежа). Выдать государственные номерные знаки, суммы и даты поступления уплаченного НДС. Выдачу отсортировать по уменьшению суммы уплаченного НДС.

```
SELECT gnz, 
       cost, 
       date$reg_certif 
FROM vehicle 
WHERE cost*0.2 > 6e5 
ORDER BY cost DESC
```
	
18. Сформировать список автомобилей, зарегистрированных в Орловской области. Вывести государственный номерной знак, серию, номер и дату выдачи свидетельства о регистрации транспортного средства. Отсортировать данные по региону регистрации, по убыванию.

```
SELECT gnz, 
       ser$reg_certif, 
       num$reg_certif, 
       date$reg_certif 
FROM vehicle 
WHERE substring(gnz from 7)::integer IN (57, 157, 757)
```
	
19. Найти работы, выполненные в выходные дни (субботу и воскресенье). Выдать государственные номерные знаки автомобилей, даты проведения работ, дни недели, в которые они проводились, и технические заключения по их результатам (tech_cond_resume). 
	
```
SELECT gnz, 
       DATE(date_work),
       to_char(date_work, 'Day') "День недели",
       tech_cond_resume 
FROM maintenance 
WHERE date_part('dow', date_work) IN (0,6)
```
	
20. Сформировать список работ, проведенных в выходные дни (кроме праздничных), по которым не сформировано техническое заключение специалиста.

```
SELECT gnz, 
       DATE(date_work),
       to_char(date_work, 'Day') "День недели",
       tech_cond_resume 
FROM maintenance 
WHERE date_part('dow', date_work) IN (0,6) 
      AND 
      tech_cond_resume IS NULL
```
	
21. Найти наименования отечественных моделей автомобилей, сформированных в соответствие с советским ГОСТ классификации и кодирования (кодировка номера модели имеет четыре разряда). 

```
SELECT model_name 
FROM model 
WHERE length(substring(model_name FROM 5))=4 
      AND 
      substring(model_name FOR 3) IN ('ГАЗ','ВАЗ','ПАЗ')
```
	
22. Выдать фамилии, инициалы механиков с фамилиями, начинающимися на буквы "А", "Ч", "Г" с упорядочением результирующей выборки по фамилии. 

```
SELECT sname_initials 
FROM mechanic
WHERE sname_initials LIKE 'А%' 
      OR sname_initials LIKE 'Ч%' 
      OR sname_initials LIKE 'Г%'
```

23. Найти автомобильные заводы, в названиях или почтовых адресах или фактических адресах которых встречается символ подчеркивания "_" (использовать предикат LIKE с конструкцией ESCAPE). Выдать названия юридических лиц, их почтовые и фактические адреса и телефоны. 

```
SELECT factory_name, 
       post_addr, 
       legal_addr, 
       phone 
FROM factory 
WHERE factory_name LIKE '%!_%' ESCAPE '!' 
      OR post_addr LIKE '%!_%' ESCAPE '!' 
      OR legal_addr LIKE '%!_%' ESCAPE '!'
```
	
24. Определить, когда последний раз проводилось обслуживание автомобиля с государственным номерным знаком 'c910ca57'. Результат представить в виде даты в формате "день.месяц.год" с указание тысячелетия (четырехразрядное обозначение года).

```
SELECT to_char(date_work,'dd.mm.yyyy') 
FROM maintenance 
WHERE gnz='c910ca57' 
ORDER BY date_work DESC 
LIMIT 1
```
	
25. Определить автомобили, которые в 2018 году посетили предприятие для обслуживания или ремонта. 

```
SELECT DISTINCT gnz 
FROM maintenance  
WHERE to_char(date_work, 'yyyy') = '2018'
```

26. Найти технические заключения, серия которых состоит только из цифр. Выдать серии, номера заключений и даты выполнения работ. 

```
SELECT s$diag_chart, 
	   mt_id, 
	   to_char(date_work, 'dd.mm.yyyy') 
FROM maintenance 
WHERE s$diag_chart ~ E'^\\d+$'
```
	
27. Найти технические заключения о проведенных работах, которые в серии имеют буквосочетание "ТО" в любом регистре, на любой позиции, выданные на работы, проведенные в 2019 году. Выдать серии и номера технических заключений через пробел в одном столбце. 

```
SELECT concat(s$diag_chart, ' ', st_id) 
FROM maintenance 
WHERE s$diag_chart ILIKE '%ТО%' 
      AND 
      extract(year FROM date_work)=2019
```
	
28. Найти работы, выполненные в последний день месяца (учитывать високосные годы). Выдать серии и номера технических заключений, даты (без указания времени) проведения работ, содержание заключения. 

```
SELECT s$diag_chart, 
       st_id, 
       to_char(date_work,'dd.mm.yyyy'), 
       tech_cond_resume 
FROM maintenance 
WHERE  date(date_work) 
       = 
       (date_trunc('month',date_work) + INTERVAL '1 MONTH - 1 DAY')::DATE
```

29. Найти автомобили, зарегистрированные за пределами Орловской области (код региона государственного номерного знака не входит во множество {57, 157, 757}). Выдать государственные номерные знаки, даты изготовления, даты начала эксплуатации, серии, номера и даты выдачи свидетельств о регистрации транспортных средств. Результат отсортировать по дате начала эксплуатации.

```
SELECT gnz, 
       date_made, 
       date_use, 
       ser$reg_certif, 
       num$reg_certif, 
       date$reg_certif 
FROM vehicle 
WHERE substring(gnz FROM 7)::integer NOT IN (57, 157, 757) 
ORDER BY date_use
```
	
30. Сформировать сортированный список государственных номерных знаков зарегистрированных автомобилей с добавлением столбца с порядковым номером записи, с названием "numrow".

```
SELECT gnz, 
       ROW_NUMBER() OVER (ORDER BY gnz)  numrow 
FROM vehicle
```	
</details>
	
<details>
<summary>Выборка из нескольких таблиц</summary>
	
31. Сформировать список производителей автомобилей и принадлежащих им заводов, отсортированный по столбцу "Производитель" по алфавиту. Столбец с названиями заводов именовать как "Завод".

```
SELECT name "Производитель", 
       factory_name "Завод"  
FROM factory f
INNER JOIN brand b ON b.idb = f.idb
ORDER BY name
```
	
32. Составить список автомобилей с указанием их государственного номерного знака (таблица vehicle), производителя (таблица brand), наименования марки (таблица marka) и модели (таблица model). Выдачу сформировать в виде двух столбцов – "Государственный номерной знак" и "Автомобиль". Во втором столбце должны быть через запятую указаны производитель, марка и модель. Учесть, что при конкатенации строк если одно из выражений возвращает NULL, то и вся строка примет значение NULL (использовать функцию COALESCE).

```
SELECT gnz AS "Государственный номерной знак", 
       COALESCE(CONCAT(b.name,', ', m.name,', ', mod.model_name),'Нет инфо') 
       "Автомобиль"
FROM vehicle v
JOIN brand b ON b.idb = v.idb
JOIN marka m ON m.idm = v.idm 
JOIN model mod ON mod.idmo = v.idmo
```
	
33. Создать список контактных телефонов производителей (телефоны заводов), по которым могут обратиться владельцы автомобилей. Указать государственный номерной знак автомобиля, наименование производителя и контактный телефон завода, на котором произведен автомобиль. 

```
SELECT v.gnz, b.name, f.phone 
FROM vehicle v
JOIN factory f ON f.idf = v.idf
JOIN brand b ON b.idb = v.idb
```
	
34. Составить список механиков, обслуживавших автомобиль с государственным номерным знаком "o009oo57". В выдачу включить дату проведения работ в формате "dd.mm.yyyy" и фамилию и инициалы механика. Результат отсортировать в хронологическом порядке.

```
SELECT to_char(date_work, 'dd.mm.yyyy'), 
       mec.sname_initials 
FROM maintenance mt
JOIN mechanic mec ON mec.id_mech = mt.id_mech
WHERE mt.gnz='o009oo57'
ORDER BY date_work DESC
```
	
35. Найти автомобили производства Японии. Указать производителя, марку, модель, разделенные пробелами в одном столбце, и государственный номерной знак. Учесть, что ряд автомобилей в атрибуте marka имеют значение NULL.

```
SELECT concat(b.name,' ', m.name,' ', mod.model_name) "Автомобиль", 
       v.gnz "Гос. номер. знак"
FROM vehicle v
JOIN brand b ON b.idb = v.idb
JOIN marka m ON m.idm = v.idm
JOIN model mod ON mod.idmo = v.idmo
JOIN state st ON st.st_id = v.st_id
WHERE st.name='Япония'
```

36. Сформировать список автомобилей, сменивших владельца (самосоединение таблицы vehicle со своей копией, совпадают даты изготовления, производители, марки, модели; различаются государственные номерные знаки, серии, номера и даты выдачи свидетельств о регистрации транспортных средств). В выдачу включить столбец "Дата изготовления", указать установленный ранее государственный номерной знак, серию, номер и дату (в формате "dd.mm.yyyy") выдачи свидетельства о регистрации транспортного средства в одном столбце, разделив пробелами. Такие же данные должны быть приведены по новому государственному регистрационному знаку и свидетельству о регистрации транспортного средства.

```
SELECT a1.date_made, 
	   a1.gnz, CONCAT_WS(' ',a1.num$reg_certif, a1.ser$reg_certif, 
	   to_char(a1.date$reg_certif, 'dd.mm.yyyy'))  "Дата изготовления",
	   b1.gnz, 
	   CONCAT_WS(' ',b1.num$reg_certif, 
	   b1.ser$reg_certif, 
	   to_char(b1.date$reg_certif, 'dd.mm.yyyy')) "Да-та изготовления"
FROM vehicle a1
JOIN vehicle b1
ON b1.date_made = a1.date_made
AND b1.gnz > a1.gnz  
JOIN brand ON brand.idb = a1.idb
JOIN marka ON marka.idm = a1.idm
JOIN model ON model.idmo = a1.idmo
```

37. Выдать список механиков (фамилии и инициалы), государственные номерные знаки обслуженных или отремонтированных ими автомобилей и даты выполнения работ с учетом возможности отсутствия выполненных заказов некоторыми механиками. 

```
SELECT m.sname_initials, 
	   mt.gnz, 
	   mt.date_work, 
	   mt.tech_cond_resume
FROM maintenance mt
RIGHT JOIN mechanic m ON mt.id_mech = m.id_mech
```
	
38. Сформировать список технических заключений по ремонтам автомобилей BMW. В выдачу включить наименование производителя, наименование завода, дату проведения ремонта без указания времени, формулировку технического заключения. Список технических заключений отсортировать по дате оформления.

```
SELECT b.name, 
	   f.factory_name, 
	   DATE(mt.date_work), 
	   mt.tech_cond_resume
FROM maintenance mt
JOIN brand b ON b.idb = mt.idb
      AND 
      b.idb = 22
JOIN factory f ON f.idf = mt.idf
WHERE mt.mt_id::INT = 20
ORDER BY mt.date_work
```

39. Найти автомобильные предприятия, расположенные на той же улице, что и "ОАО АВТОВАЗ". Выдать наименование, почтовый и фактический адрес, контактный телефон. Использовать самосоединение.

```
SELECT a1.factory_name, 
       b1.post_addr, 
       b1.legal_addr, 
       b1.phone
FROM factory a1
JOIN factory b1 ON a1.factory_name LIKE '%ОАО АВТОВАЗ%'
     AND a1.post_addr = b1.post_addr 
     AND a1.factory_name > b1.factory_name
```
	
40. Найти автомобили, которые обслуживал тот же механик, что и автомобиль с государственным номерным знаком "o929ao57". Выдать государственные номерные знаки обслуженных автомобилей, даты выполнения работ и в отдельном столбце время выполнения работ в 24-часовом формате без указания секунд.

```
SELECT b1.gnz,  
       b1.date_work, 
       to_char(b1.date_work,'hh:mm')
FROM maintenance a1
JOIN maintenance b1 ON b1.id_mech = a1.id_mech 
     AND 
     a1.gnz = 'o929ao57'
```

41. Сформировать список автомобилей, свидетельство о регистрации транспортного средства которых имеет ту же серию, что и документ автомобиля с государственным номерным знаком "c172ac57". В выдачу включить только автомобили того же производителя, что и автомобиль с государственным номерным знаком "c172ac57", указать их государственный номерной знак, наименование производителя, дату ввода в эксплуатацию (date_use).

```
SELECT t1.factory_name, 
       t1.post_addr, 
       t1.legal_addr, 
       t1.phone 
FROM factory t1 
JOIN factory t2 USING (post_addr) 
WHERE t1.idf > t2.idf 
      AND 
     t2.factory_name ~ '.ОАО АВТОВАЗ.';
SELECT t1.gnz, 
       t1.idb, 
       t1.date_use 
FROM vehicle t1 
LEFT JOIN vehicle t2 USING (ser$reg_certif) 
WHERE t2.gnz = 'c172ac57' 
      AND 
      t1.idb = t2.idb 
      AND 
      t1.gnz != t2.gnz
```
</details>

<details>
<summary>Вложенные запросы</summary>
	
42. Найти автомобили, которые никогда не обслуживались предприятием. Выдать список государственных номерных знаков этих автомобилей.

```
SELECT v.gnz 
FROM vehicle v
WHERE v.gnz NOT IN (SELECT gnz 
                    FROM maintenance)
```
	
43. Составить список автомобилей (государственный номерной знак и стоимость), которые стоят не более средней стоимости всех зарегистрированных автомобилей.

```
SELECT gnz, 
       cost
FROM vehicle 
WHERE cost<=(SELECT AVG(cost) 
	     FROM vehicle)
```
		   
44. Найти автомобили, которые были приобретены не новыми. К таким можно отнести экземпляры, у которых год и месяц начала эксплуатации и год и месяц даты выдачи свидетельства о регистрации транспортного средства не совпадают.

```
SELECT gnz
FROM vehicle 
WHERE (SELECT to_char(date_use, 'yyyy.mm')) 
       != 
      (SELECT to_char(date$reg_certif, 'yyyy.mm'))
```
		   
45. Найти автомобили, изготовленные на том же заводе, что и автомобиль с государственным номерным знаком "x027kp57". Выдать их государственные номерные знаки, наименование, почтовый адрес и контактный телефон завода. 

		   
```
SELECT v.gnz, 
       f.factory_name, 
       f.post_addr, 
       f.phone
FROM factory f
JOIN vehicle v ON v.idf = f.idf
     AND 
     (SELECT vehicle.idf 
     FROM vehicle 
     WHERE vehicle.gnz='x027kp57')
     =
     f.idf 
     AND v.gnz != 'x027kp57'
```
		   
46. Составить список автомобильных брендов, не имеющих собственного производства на территории Российской Федерации. Указать их наименования, государственную принадлежность.

		   
```
SELECT b.name, 
       st.name 
FROM state st
JOIN brand b ON b.st_id = st.st_id
AND 
(SELECT state.st_id 
FROM state 
WHERE state.name='Российская Федерация') != b.st_id
```
		   
47. Найти производителей, которые имеют заводы, как на территории Российской Федерации, так и за ее пределами. Указать наименование бренда, название и адрес размещения завода.

```
WITH 
rus_br AS
(
SELECT idb
FROM factory
WHERE st_id != 1
	  AND 
	  legal_addr ILIKE '%Россия%'
)
SELECT b.name,
	   f.factory_name,
	   f.legal_addr
FROM brand b 
JOIN factory f ON f.idb = b.idb
WHERE b.idb IN (SELECT *
		FROM rus_br
	       )
```

48. Определить почтовый адрес завода, изготовившего автомобиль с государственным номерным знаком "a723ak57", для направления претензии по недостатку, выявленному в ходе проведения ремонта 6 ноября 2018 года. В выдачу включить государственный номерной знак, производителя, марку и модель автомобиля в одной колонке через запятую, дату изготовления автомобиля, наименование завода-изготовителя, его почтовый адрес, дату проведения ремонта, серию и номер выданной диагностической карты в одной колонке через пробел, техническое заключение по ремонту.


```
SELECT mt.gnz, 
	   concat_ws(', ',b.name, m.name, md.model_name), 
	   v.date_made,
	   f.factory_name, 
	   f.post_addr, 
	   mt.date_work, 
	   concat_ws(' ',mt.s$diag_chart, mt.n$diag_chart),
	   mt.tech_cond_resume
FROM maintenance mt
INNER JOIN vehicle v ON v.gnz 
		IN (
    	SELECT gnz 
    	FROM maintenance
        WHERE gnz = 'a723ak57')
	    AND mt.date_work::DATE = '06-11-2018'
JOIN marka m ON m.idm = mt.idm
JOIN brand b ON b.idb = mt.idb
JOIN model md ON md.idmo = mt.idmo
JOIN factory f ON f.idf = mt.idf
```

49. Рассчитать количество заказов по видам работ. Выдачу сформировать в виде таблицы, где предусмотреть три столбца: "Техническое обслуживание", включив в подсчет все виды технического обслуживания; "Ремонт"; "Предпродажная подготовка". 

```
SELECT 
(SELECT COUNT(date_work) 
FROM maintenance 
WHERE mt_id::INT BETWEEN 1 AND 18) "Техническое обслуживание",
(SELECT COUNT(date_work) 
FROM maintenance 
WHERE mt_id::INT = 20) "Ремонт", 
(SELECT COUNT(date_work) 
FROM maintenance 
WHERE mt_id::INT = 19) "Предпродажная подготовка"
FROM maintenance
LIMIT 1
```

50. Найти механиков, которые выполнили 2 и более заказов в один день. Выдать их фамилии и инициалы.


```
SELECT m.sname_initials
FROM maintenance mt
JOIN mechanic m ON mt.id_mech = m.id_mech
GROUP BY mt.date_work::Date, m.sname_initials
HAVING COUNT(date_work::date) >= 2
```
	
</details>	

<details>
<summary>Теоретико-множественные операции</summary>
	
51. Найти автомобили, претендующие на отнесение к классу раритетных. К таковым относят автомобили отечественного производства в возрасте не менее 30 лет, либо зарубежные автомобили в возрасте не менее 25 лет, либо автомобили, имеющие пробег не менее 500000 км без учета возраста. Указать государственный номерной знак, год выпуска и пробег каждого из них.

```
SELECT v.gnz,
       EXTRACT(YEAR FROM v.date_made) god,
       v.run
FROM vehicle v
JOIN brand b ON b.idb = v.idb
WHERE (b.idb IN (1, 2, 8, 9)
       AND
       DATE_PART('YEAR', AGE(CURRENT_DATE, v.date_made))::INT >= 30)
       OR
       (b.idb IN (22,23,11,31,32,41,42)
       AND 
       DATE_PART('YEAR', AGE(CURRENT_DATE, v.date_made))::INT >= 25)
       OR
       (v.run > 500000)
```

52. Найти автомобили, которые посещали предприятие только по пятницам. Выдать государственные номерные знаки. 

```
SELECT DISTINCT m.gnz
FROM maintenance m
WHERE EXTRACT(DOW FROM date_work) = 5
	  AND
	  NOT EXISTS	
	  			(SELECT gnz
				FROM maintenance mt
				WHERE m.gnz = mt.gnz
				 	  AND
				      EXTRACT(DOW FROM date_work) != 5)
```
	
53. Найти все автомобили, обслуженные механиком Баженовым М.К. (все виды ТО), и (в том числе включительно) отремонтированные механиком Савостьяновым А.В. (только ремонты). Указать их государственные номерные знаки.

```
WITH
baz AS
(
	    SELECT gnz
	    FROM maintenance
	    WHERE 
		(id_mech = 5
		AND 
		mt_id::INT BETWEEN 1 AND 18)
),
sev AS
(
	SELECT gnz
	FROM maintenance
	WHERE id_mech = 1
	      AND
	      mt_id::INT = 20
)
SELECT baz.gnz
FROM baz
JOIN sev ON baz.gnz = sev.gnz
```
	
54. Найти механиков, которые в 2018 году ежемесячно (без пропусков) получали наряды на обслуживание или ремонт автомобилей. Выдать их фамилии и инициалы.

```
(SELECT first_value(sname_initials) OVER (ORDER BY 1) sname 
FROM mechanic AS m1
JOIN maintenance m ON m1.id_mech = m.id_mech
WHERE extract('year' from date_work) = 2018 
GROUP BY m1.id_mech, m1.sname_initials 
HAVING COUNT(DISTINCT extract('month' from date_work)) = 12)
```
	
55. Найти автомобили, которые обслуживались только в 2018 году. Указать государственный номерной знак, дату проведения обслуживания и техническое заключение по его результатам.

```
SELECT gnz,
       date_work dt,
       tech_cond_resume tcr
FROM maintenance m
WHERE EXTRACT(YEAR FROM date_work::date) = 2018
	  AND
	  NOT EXISTS
	  		(SELECT gnz
			FROM maintenance mt
			WHERE m.gnz = mt.gnz
			AND
			EXTRACT(YEAR FROM date_work::date) != 2018
			)
```

56. Выдать список рабочих дней в феврале 2018 года, в которые не выполнялись заказы по обслуживанию или ремонту автомобилей. Выдать даты дней без заказов.

```
WITH
gendt AS
(
SELECT i::date 
FROM generate_series('2018-02-01',
		     '2018-02-28',
		     '1 day'::INTERVAL) i
)
SELECT gendt.i
FROM gendt
LEFT JOIN maintenance mt ON mt.date_work::date = gendt.i
WHERE mt.date_work IS NULL
	  AND
	  EXTRACT(ISODOW FROM gendt.i) NOT IN (6,7)
	  AND gendt.i <> '2018-02-23'	
```
</details>	

<details>
<summary>Агрегирование данных, групповые операции</summary>
	
57. Определить количество работ, выполненных в 2017 году. 

```
SELECT COUNT(gnz) 
FROM maintenance 
WHERE EXTRACT(year FROM date_work)=2017
```
	
58. Рассчитать общую сумму НДС, уплаченную в 2016 году (НДС рассчитывается как 18% от суммы платежа) за приобретенные автомобили. Результат округлить до копеек и представить в виде количества рублей и копеек. 

```
SELECT to_char(SUM(cost)*0.18,'999 999 999 руб. 99 коп')
FROM vehicle 
WHERE EXTRACT(year FROM date$reg_certif)=2016
```
	
59. Определить, сколько учтено автомобилей, зарегистрированных в Орловской области. 

```
SELECT COUNT(gnz) 
FROM vehicle
WHERE SUBSTRING(gnz FROM 7 FOR 2)::INT IN ( 57 )
```
	
60. Определить средний возраст механиков предприятия с точностью до двух значащих цифр мантиссы.

```
SELECT ROUND(((EXTRACT(YEAR FROM AVG(AGE(born))) * 12 
       + 
       EXTRACT(MONTH FROM AVG(AGE(BORN))))/12), 2) 
FROM mechanic
```
	
61. Определить общую и среднюю стоимость с точностью до копейки, общий и средний пробег с точностью до 100 м всех зарегистрированных автомобилей. Указать в качестве имен столбцов требуемые вычисления.

```
SELECT ROUND(SUM(cost), 2)  "Общая стоимость",
       ROUND(AVG(cost), 2) "Средняя стоимость",
       ROUND(SUM(run), 1) "Общий пробег",
       ROUND(AVG(run), 1) "Средний пробег"
FROM vehicle
```
	
62. Определить средний пробег автомобилей каждого бренда. Результат округлить до 10 м. 

```
SELECT ROUND(AVG(vehicle.run),2),
       b.name
FROM vehicle
JOIN brand b ON b.idb = vehicle.idb
GROUP BY vehicle.idb, b.name
```

63. Рассчитать среднюю стоимость с точностью до копейки каждой марки зарегистрированных автомобилей. В выдачу включить наименование бренда, марки и среднюю стоимость.

```
SELECT AVG(v.cost)::money,
       b.name,
       m.name
FROM vehicle v
JOIN brand b ON b.idb = v.idb
JOIN marka m ON m.idm = v.idm
GROUP BY  b.name, m.name
```
	
64. Определить с точностью до двух значащих цифр мантиссы средний возраст автомобилей каждой марки. Для автомобилей, у которых не предусмотрена марка, указывать модель.

```
SELECT DISTINCT ON (m.name) ROUND(CAST(AVG(DATE_PART('YEAR', AGE(v.date_made)) )AS NUMERIC), 2),
	CASE
	   WHEN m.name IS NULL THEN  md.model_name
	   ELSE m.name
	END
FROM vehicle v
JOIN marka m ON m.idm = v.idm
JOIN model md ON v.idmo = md.idmo
GROUP BY m.name, md.model_name
```
	
65. Определить год, за который поступило больше всего заказов (относительно других лет).

```
SELECT EXTRACT(year FROM date_work::date) years
FROM maintenance
GROUP BY EXTRACT(year FROM date_work::date)
ORDER BY COUNT(date_work) DESC
LIMIT 1
```

66. Построить распределение марок автомобилей, ограничив список марками, встречающимися не менее 8 раз. Список упорядочить по уменьшению количества экземпляров марки.

```
SELECT CONCAT(b.name, ' ',m.name),
       COUNT(m.name)
FROM vehicle v
JOIN marka m ON m.idm = v.idm
JOIN brand b ON b.idb = v.idb
GROUP BY b.name, m.name
HAVING COUNT(m.name) >= 8
ORDER BY 2 DESC
```
	
67. Найти автомобили, владельцы которых за все время разместили заказ только один раз. Выдать государственные номерные знаки.

```
SELECT gnz
FROM maintenance
GROUP BY gnz
HAVING COUNT(date_work) = 1
```
	
</details>
	
<details>
<summary>Совместное использование конструкций языка SQL</summary>
	
68. Найти автомобили, выпущенные в Евросоюзе. Выдать государственные номерные знаки, государственную принадлежность и наименование завода-изготовителя, его фактический адрес и телефон.

```
SELECT v.gnz,
       s.name,
       f.factory_name,
       f.legal_addr,
       f.phone
FROM vehicle v
JOIN state s ON s.st_id = v.st_id
JOIN factory f ON f.idf = v.idf
WHERE s.st_id IN (2,4)
```

69. Найти автомобили, которые проходили на предприятии только предпродажную подготовку. Указать их государственные номерные знаки, дату предпродажной подготовки, фамилию и инициалы механика, проводившего работы.

```
SELECT mt.gnz,
       mt.date_work::DATE,
       m.sname_initials
FROM maintenance mt
JOIN mechanic m ON m.id_mech = mt.id_mech
JOIN maintenancetype mtt ON mtt.mt_id = mt.mt_id
WHERE mtt.mt_id::int = 19
```

70. Определить автомобильный бренд, на который клиенты предприятия, вместе потратили больше всех денег (найти «автомобиль богатых»).

```
WITH 
br AS
(
SELECT idb,
       SUM(cost) totalc
FROM vehicle
GROUP BY idb
ORDER BY totalc DESC
LIMIT 1
)
SELECT b.name
FROM brand b
JOIN br ON b.idb = br.idb
```

71. Определить, сколько автобусов обслужено механиком Кротовым К.О.

```
SELECT COUNT(mt.id_tg)
FROM maintenance mt
WHERE mt.id_mech = (SELECT id_mech
		    FROM mechanic
		    WHERE sname_initials ILIKE '%Кротов К.О.%')
	 	    AND
	 	    mt.id_tg = (SELECT id_tg
				FROM transpgroup
				WHERE name ILIKE '%Автобус%')
```
	
72. Найти автомобили, которые были приобретены не новыми (интервал между датой выдачи свидетельства о регистрации транспортного средства и датой начала эксплуатации больше двух недель). Выдать государственные номерные знаки, производителя, марку, модель, серию, номер и дату выдачи свидетельства о регистрации транспортного средства, дату начала эксплуатации. Все данные, кроме даты начала эксплуатации организовать одним столбцом по формату: <Государственный номерной знак><Производитель><Марка><Модель>, Свидетельство о регистрации <Серия СРТС> № <Номер СРТС> выдано: <Дата выдачи СРТС>.

```
SELECT v.date_use,
       CONCAT(v.gnz, 
              b.name,
              mr.name,
              md.model_name,
              'Свидетельство о регистрации: ',v.ser$reg_certif,
              '№',v.num$reg_certif,
              'выдано: ', v.date$reg_certif
              )
FROM vehicle v
JOIN brand b ON b.idb = v.idb
JOIN marka mr ON mr.idm = v.idm
JOIN model md ON md.idmo = v.idmo
WHERE AGE(date$reg_certif, date_use) > '14 days'
```
	
73. Сформировать список заводов по производству автомобилей, размещенных на территории Российской Федерации, и, в зависимости от того, входит ли страна бренда в Европейский союз или нет, указать наименование бренда, предприятия, почтовый или фактический адрес соответственно (для стран Евросоюза указывать почтовый адрес), телефон.

```
SELECT b.name,
       f.factory_name,
	   CASE
	   		WHEN f.st_id IN (2,4) THEN f.post_addr
	   		ELSE f.legal_addr
	   END post_addr,
       f.phone
FROM brand b
JOIN factory f ON b.idb = f.idb 
WHERE f.legal_addr ILIKE '%Россия%'
```
	
74. Найти производителей, автомобили которых в 2018 году реже остальных требовали ремонта. Выдать названия брендов и количество ремонтов их автомобилей.

```
SELECT b.name,
       COUNT(mt.mt_id) "Количество ремонтов"
FROM maintenance mt
JOIN brand b ON b.idb = mt.idb
JOIN maintenancetype mtt ON mtt.mt_id = mt.mt_id
WHERE mtt.mt_id::int = 20
      AND
      EXTRACT(YEAR FROM mt.date_work) = 2018
GROUP BY mt.mt_id, b.name
ORDER BY 2 ASC
LIMIT 1
```

75. Найти механиков, которые выполнили больше работ, чем Голубев Д.Н. В выдачу включить фамилии и инициалы этих людей.

```
SELECT mec.sname_initials
FROM maintenance mt
JOIN mechanic mec ON mt.id_mech = mec.id_mech
WHERE mt.mt_id::INT BETWEEN 1 AND 18
GROUP BY 1
HAVING COUNT(date_work) > (SELECT 
	   		   COUNT(date_work) cdw
			   FROM maintenance
			   WHERE id_mech = 12
			   AND
			   mt_id::INT BETWEEN 1 AND 18
			   )
```
	
76. Найти автомобили, зарегистрированные в один и тот же день. Выдать государственные номерные знаки, в одном столбце через пробел производителя, марку и модель каждого из них, дату регистрации.
	
```
SELECT v.gnz,
       CONCAT_WS(' ',br.name, m.name, mod.model_name),
       v.date$reg_certif
FROM vehicle v
JOIN vehicle b ON b.date$reg_certif = v.date$reg_certif
JOIN brand br ON br.idb = v.idb
JOIN marka m ON m.idm = v.idm
JOIN model mod ON mod.idmo = v.idmo
WHERE v.gnz != b.gnz
```

77. Для каждого автомобиля указать число посещения им предприятия (учитывать, что могут быть автомобили, которые ни разу не обслуживались, в этом случае выводить значение 0). Вывести государственные номерные знаки, серии, номера и даты их свидетельств о регистрации транспортного средства и количество посещений. Выдачу отсортировать по количеству посещений.

```
SELECT  v.gnz,
	v.ser$reg_certif,
	v.num$reg_certif,
	v.date$reg_certif,
	COUNT(mt.date_work) totalW
FROM maintenance mt
RIGHT JOIN vehicle v ON mt.gnz = v.gnz
GROUP BY 1,2,3,4
ORDER BY totalW DESC
```
	
78. Найти автомобили, которые в 2016, 2017 и 2018 годах совершили 80% и более посещений предприятия от всего объема их обслуживания за все время. Вывести их государственные номерные знаки. 

```
SELECT m.gnz, 
       COUNT(to_char(m.date_work, 'yyyy')) 
FROM maintenance m,
    (SELECT m.gnz, COUNT(to_char(m.date_work, 'yyyy')) 
     FROM maintenance m 
     WHERE to_char(date_work, 'yyyy') = '2016' OR to_char(date_work, 'yyyy') = '2017' 
     OR to_char(date_work, 'yyyy') = '2018' GROUP BY m.gnz) 
TEMP WHERE m.gnz = TEMP.gnz GROUP BY m.gnz, TEMP.count 
HAVING COUNT(to_char(m.date_work, 'yyyy'))*0.8 <= TEMP.count
```

79. Найти механиков, получивших сертификат на работу после достижения ими пенсионного возраста. Учесть, что до 2018 года возраст выхода на пенсию для мужчин составлял 60, а для женщин – 55 лет, а с 2018 года эти показатели увеличены на 5 лет и действуют относительно тех, кому настал срок выхода на пенсию. Прогрессивную шкалу роста пенсионного возраста не учитывать. Выдать фамилии, инициалы и даты рождения механиков, даты получения ими сертификатов и приема на работу.

```
SELECT sname_initials,
       born,
       certif_date,
       work_in_date
FROM mechanic
WHERE DATE_PART('YEAR',AGE(born)) >= 65
OR 
DATE_PART('YEAR',AGE(born)) >= 60
```
	
80. Сформировать отчет о выполненных ремонтах автомобилей за все время работы предприятия. В отчете отобразить: государственный номерной знак; в одном столбце через запятую наименование производителя, марку и модель; также в одном столбце указать через пробел серию, номер и дату выдачи свидетельства о регистрации транспортного средства; дату проведения ремонта; фамилию и инициалы механика, выполнившего ремонт; техническое заключение по ремонту. Все даты приводить в формате "dd.mm.yyyy".

```
SELECT mt.gnz,
       CONCAT_WS(', ',f.factory_name, m.name, mod.model_name),
       CONCAT_WS(' ',v.ser$reg_certif, num$reg_certif, to_char(date$reg_certif,'dd.mm.yyyy')),
       to_char(mt.date_work,'dd.mm.yyyy'),
       mec.sname_initials,
       mt.tech_cond_resume
FROM maintenance mt
JOIN factory f ON f.idf = mt.idf 
JOIN marka m ON m.idm = mt.idm
JOIN model mod ON mod.idmo = mt.idmo
JOIN vehicle v ON mt.gnz = v.gnz
JOIN mechanic mec ON mec.id_mech = mt.id_mech
```
	
81. Определить долю в процентах (с точностью до двух значащих цифр мантиссы) в общем результате предприятия механика Савостьянова А.В. Считать, что все работы (заказы на ремонт или обслуживание) являются одинаково весомыми в общих итогах работы предприятия. 

```
WITH
sev AS
(
SELECT m.id_mech mec,
       COUNT(mt.date_work) cmt
FROM mechanic m
JOIN maintenance mt ON mt.id_mech = m.id_mech 
WHERE m.sname_initials ILIKE '%Савостьянов А.В.%'
GROUP BY 1
)
SELECT ROUND((cmt / (SELECT COUNT(date_work)
		     FROM maintenance)::numeric) * 100, 2)
FROM sev
```
	
82. Сформировать список инвестиционно не выгодных автомобилей. К таковым относятся автомобили с пробегом не менее 100 000 км, или имеющие возраст 3 и более года, или побывавшие в ремонте хотя бы один раз, а также автомобили из транспортных групп "Специальные автомобили", "Специализированные автомобили", "Спортивные автомобили" или "Спортивные мотоциклы". В список включить столбцы: "Государственный номерной знак", "Возраст", "Пробег" и "Дата последнего ремонта". Если автомобиль в ремонте не был, то в последнем столбце должен храниться пробел. 

```	
WITH
remont AS
(
	SELECT gnz,
	       COUNT(date_work) cdw
	FROM maintenance
	GROUP BY gnz
),
spec_id AS
(
	SELECT id_tg
	FROM transpgroup
	WHERE name ILIKE ANY (ARRAY['%Специальные автомо-били%',
					  '%Специализированные автомобили%',
					  '%Спортивные автомобили%',
					  '%Спортивные мотоциклы%'])
)
SELECT v.gnz "Гос. номерной знак",
	   DATE_PART('YEAR',AGE(v.date_made)) "Возраст",
	   v.run "Пробег",
	   CASE
	   		WHEN to_char(MAX(mt.date_work),'dd-mm-yyyy') IS NOT NULL THEN to_char(MAX(mt.date_work),'dd-mm-yyyy')
	   		ELSE COALESCE(to_char(MAX(mt.date_work::DATE),'dd-mm-yyyy'),'') 
	   END "Дата последнего ремонта"
FROM vehicle v
LEFT JOIN remont ON remont.gnz = v.gnz
LEFT JOIN spec_id ON spec_id.id_tg = v.id_tg
LEFT JOIN maintenance mt ON mt.gnz = v.gnz
WHERE v.run >= 100000
	  OR
	  DATE_PART('YEAR',AGE(v.date_made)) >= 3
	  OR
	  remont.cdw >= 1
	  OR
	  v.id_tg = spec_id.id_tg
GROUP BY v.gnz, v.date_made, v.run
```
	
83. Определить проводилось ли не регламентное техническое обслуживание автомобилей японского производства. Не регламентным считается любое техническое обслуживание, не предусмотренное для автомобилей, выпущенных японскими производителями. В выдаче указать государственные номерные знаки, производителя, марку, модель автомобиля, вид, дату и заключение по проведенному не регламентному ТО, фамилию и инициалы механика, выполнявшего работы.

```
SELECT DISTINCT ON (mt.gnz) 
       mt.gnz,
       b.name,
       m.name,
       mtt.name,
       mod.model_name,
       mt.date_work::DATE,
       mt.tech_cond_resume,
       mec.sname_initials
FROM maintenance mt
JOIN maintenancetype mtt ON mt.mt_id = mtt.mt_id
JOIN brand b ON b.idb = mt.idb
JOIN marka m ON mt.idm = m.idm
JOIN model mod ON mod.idmo = mt.idmo
JOIN mechanic mec ON mec.id_mech = mt.id_mech
WHERE mt.st_id = 6
AND
mtt.mt_id::INT BETWEEN 1 AND 10
```
</details>	
	
<details>
<summary>Задания повышенной сложности</summary>
	
84. Определить самый не надежный автомобиль, который имеет наименьший интервал между двумя любыми ремонтами. Указать его государственный номерной знак и наименьший интервал между ремонтами в секундах.

```
SELECT m1.gnz, 
       EXTRACT(EPOCH FROM AGE(m1.date_work, m2.date_work)) 
FROM maintenance m1 
CROSS JOIN maintenance m2 
WHERE m1.gnz = m2.gnz 
      AND 
      m1.date_work > m2.date_work 
ORDER BY AGE(m1.date_work, m2.date_work) 
LIMIT 1
```

85. Найти объем убыли клиентов с ростом возраста автомобилей, составив таблицу, где в одном столбце указан номер ТО, а в другом – число выполненных работ соответствующего вида. Данные должны быть отсортированы по номеру и виду ТО, сначала ТО-1. После перечисления всех видов ТО приводятся сведения по ТО для японских автомобилей.

```
SELECT mtt.name,
       COUNT(m.date_work)
FROM maintenance m
JOIN maintenancetype mtt ON m.mt_id = mtt.mt_id
WHERE mtt.mt_id::int NOT IN (1,2,19,20)
GROUP BY 1
ORDER BY array_position(array['ТО-1','ТО-2','ТО-3','ТО-4','ТО-5'
		 ,'ТО-6','ТО-7','ТО-8',
		 'ТО-1 для японских автомобилей']::varchar[]
		 , mtt.name)
```
	
86. Составить таблицу изменения рентабельности предприятия по годам, где показаны абсолютное число выполненных заказов, относительное число заказов на один зарегистрированный автомобиль (учесть, что после выполнения предпродажной подготовки, автомобиль более не является зарегистрированным, хотя данные о нем сохраняются в базе данных), абсолютный прирост числа заказов, упущенная выгода в виде не добранных процентов если считать за 100% ситуацию, когда все зарегистрированные автомобили прибывают на предприятие один раз в год.

```
WITH years AS 
(SELECT DISTINCT EXTRACT(YEAR FROM date_work) AS year 
FROM maintenance)
SELECT year,
       (SELECT COUNT(*) AS n_orders FROM maintenance WHERE EXTRACT(YEAR FROM date_work) = year),
       (SELECT COUNT(*) AS n_orders FROM maintenance WHERE EXTRACT(YEAR FROM date_work) = year)::numeric /
       (SELECT COUNT(*) AS n_auto
        FROM (SELECT gnz
              FROM vehicle
              EXCEPT
              SELECT gnz
              FROM maintenance
              WHERE mt_id = '19'
                AND extract(YEAR FROM date_work) <= years.year) AS t)::numeric                      AS ratio,
       (SELECT COUNT(*) AS n_orders FROM maintenance WHERE EXTRACT(YEAR FROM date_work) = year) -
       (SELECT COUNT(*) AS n_orders FROM maintenance WHERE EXTRACT(YEAR FROM date_work) = year - 1) AS growing,
       (SELECT SUM(t.c) / COUNT(t.c)
        FROM (SELECT COALESCE(t.c, 0) AS c
              FROM vehicle AS v
                       LEFT JOIN (SELECT mt.gnz, COUNT(*) AS c
                                  FROM maintenance AS mt
                                  WHERE NOT EXISTS(SELECT *
                                                   FROM maintenance AS mt2
                                                   WHERE mt2.mt_id = '19'
                                                     AND mt.gnz = mt2.gnz
                                                     AND EXTRACT(year from date_work) <= year)
                                    AND EXTRACT(year from date_work) = year
                                  GROUP BY mt.gnz) AS t USING (gnz)) AS t) 
		AS lost_profit
FROM years
ORDER BY year
```

87. Составить "возрастную карту" зарегистрированных автомобилей, включив в нее столбец наименований изготовителей, столбцы для указания доли в процентах, округленной до двух значащих цифр мантиссы, автомобилей в возрасте от 0 до 6 лет, от 7 до 10 лет, от 11 до 13 лет, от 14 до 18 лет и старше 18 лет.

```
WITH
t0 AS
(
SELECT b.name,
	   DATE_PART('YEAR',AGE(v.date_made)),
	   CASE
	   	WHEN DATE_PART('YEAR',AGE(v.date_made)) BETWEEN 0 AND 6 THEN COUNT(v.gnz) 
	   END	 s06,
	   CASE
	   	WHEN DATE_PART('YEAR',AGE(v.date_made)) BETWEEN 7 AND 10 THEN COUNT(v.gnz) 
	   END	 s710,
	   CASE
	   	WHEN DATE_PART('YEAR',AGE(v.date_made)) BETWEEN 11 AND 13 THEN COUNT(v.gnz) 
	   END	 s1113,
	   CASE
	   	WHEN DATE_PART('YEAR',AGE(v.date_made)) BETWEEN 14 AND 18 THEN COUNT(v.gnz) 
	   END	 s1418,
	   CASE
	   	WHEN DATE_PART('YEAR',AGE(v.date_made)) > 18 THEN COUNT(v.gnz) 
	   END	 s18
FROM brand b
JOIN vehicle v ON v.idb = b.idb
GROUP BY b.name,v.date_made
),
done AS
(
SELECT t0.name,
	   (SUM(s06)
	   /
	   (SELECT COUNT(gnz)
		FROM vehicle) * 100) y06,
	   SUM(s710)
	   /
	   (SELECT COUNT(gnz)
		FROM vehicle) * 100 y710, 
	   SUM(s1113)
	   /
	   (SELECT COUNT(gnz)
		FROM vehicle) * 100 y1113, 
	   SUM(s1418)
	   /
	   (SELECT COUNT(gnz)
		FROM vehicle) * 100 y1418,
	   SUM(s18)
	   /
	   (SELECT COUNT(gnz)
		FROM vehicle) * 100 y18
FROM t0
GROUP BY t0.name
)
SELECT name "Изготовитель",
	   ROUND(y06,2) "0-6 лет",
	   COALESCE(ROUND(y710,2),0) "7-10 лет",
	   COALESCE(ROUND(y1113,2),0) "11-13 лет",
	   COALESCE(ROUND(y1418,2),0) "14-18 лет",
	   COALESCE(ROUND(y18,2),0) "Больше 18 лет"
FROM done
```
	
88. Определить завод-изготовитель, продукция которого больше других требует ремонта (гарантийный срок не учитывать) в абсолютных показателях и завод с наибольшей долей отказов продукции (число ремонтов на один зарегистрированный в базе данных автомобиль). Выдать наименования, принадлежность брендам, страны брендов, почтовые адреса и телефоны (в двух столбцах), количество ремонтов выпущенных ими автомобилей и долю ремонтов на один зарегистрированный автомобиль.

```
SELECT factory_name, b.name, s.name, f.post_addr, f.phone,
       (SELECT COUNT(*) FROM maintenance AS mt WHERE mt.idf = f.idf),
       (SELECT (SELECT COUNT(*) FROM vehicle AS v WHERE f.idf = v.idf) /
               (SELECT COUNT(*) FROM maintenance AS mt WHERE mt.idf = f.idf)::numeric
        WHERE (SELECT COUNT(*) FROM vehicle AS v WHERE f.idf = v.idf) != 0) bb
FROM factory f 
JOIN state s ON f.st_id = s.st_id 
JOIN brand b on b.idb = f.idb 
WHERE (SELECT COUNT(*) FROM vehicle AS v WHERE f.idf = v.idf) != 0 
ORDER BY bb DESC 
LIMIT 1
```

89. Найти автомобили с заводским браком (интервал времени между датой регистрации и первым ремонтом, не превышающий 1 года). Выдать их государственные номерные знаки; производителя, марку и модель в одном столбце; дату регистрации; дату первого ремонта; интервал в днях от регистрации до первого ремонта.

```
SELECT gnz,
       concat(b.name, ' ', m.name, ' ', mo.model_name),
       date$reg_certif,
       (SELECT date_work
        FROM maintenance
        WHERE vehicle.gnz = maintenance.gnz
          AND maintenance.mt_id = '20'
        ORDER BY date_work
        LIMIT 1),
       (EXTRACT(EPOCH FROM AGE((SELECT date_work
            FROM maintenance
            WHERE vehicle.gnz = maintenance.gnz
              AND maintenance.mt_id = '20'
            ORDER BY date_work
            LIMIT 1), date$reg_certif))/3600/24)::int AS interval
FROM vehicle
JOIN brand b ON b.idb = vehicle.idb
JOIN marka m ON vehicle.idm = m.idm
JOIN model mo ON vehicle.idmo = mo.idmo
WHERE AGE((SELECT date_work
           FROM maintenance
           WHERE vehicle.gnz = maintenance.gnz
             AND maintenance.mt_id = '20'
           ORDER BY date_work
           LIMIT 1), date$reg_certif) < '1 year'::interval
```

90. Определить медианное значение и разброс стоимости зарегистрированных автомобилей, считая, что стоимость распределена нормально. Для определения медианного значения стоимости использовать математическое ожидание, рассчитанное, как сумма произведений каждой стоимости на количество ее повторов в ряду стоимостей, деленное на общее число зарегистрированных автомобилей. Разброс рассчитать, как квадратный корень из разности медианы ранжированного ряда квадратов стоимости и квадрата медианы.

```
SELECT SUM(TRUNC((((TEMP.cost * TEMP.count)/161)::numeric),0))::money "Медиана", 
       TRUNC(CEILING((|/(SUM(TRUNC((((TEMP.cost^2 * TEMP.count)/161)::numeric),0)) - SUM(TRUNC((((TEMP.cost * TEMP.count)/161)::numeric),0))^2)))::numeric,0)::money "Разброс"
FROM
(SELECT v.cost, count(v.cost)
FROM vehicle v
GROUP BY v.cost
ORDER BY COUNT(v.cost) DESC) TEMP
```
						  
</details>
