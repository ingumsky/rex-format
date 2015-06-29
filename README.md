Формат обмена данными об объектах недвижимости
==============================================

Здесь описывается первая версия XML формата файлов, служащих для обмена информацией о
сделках с объектами недвижимости.

В данном репозитории находятся:

 - пример файла в формате REX ([rex-example.xml](rex-example.xml))
 - схема Relax NG для проверки валидности XML ([rex-schema.rnc](rex-schema.rnc) и [rex-schema.rng](rex-schema.rng))
 - файл со словарями, на которые ссылаются словарные элементы в XML ([dic.xml](dic.xml))

О терминах и названиях
----------------------

Описываемый формат называется "REX" (real estate exchange). Синоним - "наш формат".

Импорт - это преобразование из чужого формата в наш.

Экспорт - преобразование из нашего формата в чужой.

Продажа - это когда клиент продаёт свой объект через нас.

Покупка - когда клиент хочет что-то купить через нас (фактически, это задача по поиску недвижимости  по заданным критериям).

Аренда - когда клиент хочет что-то арендовать через нас.

Сдача в аренду - когда клиент хочет сдать свой объект через нас в аренду.

Рекламное издание - журнал, сайт, газета или другой ресурс, который публикует (обычно за деньги) информацию об объектах недвижимости. Принимает данные в структурированном виде.

Рекламный источник - то же, что и издание. С нашей точки зрения - это место, откуда можно взять информацию об объектах недвижимости в каком-то структурированном виде.

Реклама - это отправка агентством недвижимости данных о своих объектах в то или иное издание. Соответственно, реклама касается, в первую очередь, сделок по продаже и сдаче в аренду.

Импорт из источника - это получение откуда-нибудь структурированной информации об объектах недвижимости и сохранение в базе данных нашего проекта с целью дальнейшего анализа.

Единицы измерения
-----------------

Если не указано иное, то всегда: расстояние - в километрах, периоды времени - в месяцах, площадь - в квадратных метрах, денежные суммы - в рублях, физические размеры объектов - в метрах, размеры файлов и данных - в байтах.

Отдельно следует заметить, что площади земельных участков - тоже в квадратных метрах (для единообразия). В большинстве источников эти площади указываются в сотках или гектарах. Сотка - это 100 квадратных метров. Гектар - это 10000 квадратных метров. Эти площади при экспорте и импорте нужно будет умножать или делить.

Общая структура файла
---------------------

Корневой элемент - root, единственный его обязательный атрибут  - rev, определяющий версию формата. Данные о сделках хранятся в коллекции orders, каждая сделка описывается одним элементом order. Атрибуты: id - номер сделки в нашем проекте, ctm - unix timestamp времени создания сделки, mtm - unix timestamp времени последнего изменения сделки. Все атрибуты необязательны.

В элемент order вложены элементы, описывающие сделку. Эти элементы могут идти в произвольном порядке; практически все могут отсутствовать. Единственное исключение - элемент type, определяющий отношения компании и клиента (продажа, покупка, аренда, сопровождение, etc). Без этих отношений сделки не может быть, поэтому данный элемент обязателен. Значение - одно из словаря order_type.

Ниже приведён общий вид XML файла в формате REX. Он технически некорректен (корректный пример - в отдельном файле), зато в нём приведены краткие комментарии по поводу каждого элемента, присутствующего в order.

	<root rev="1">
		<orders>
			<order id="13234" ctm="1281381923" mtm="12391820398">
				<title>Название сделки</title>
				<type>Тип сделки: продажа, аренда, покупка, etc</type>
				<group>Данные о группе объектов</group>
				<price>
					Цена продажи, аренды, etc. Зависит от типа сделки. Для аренды указывается срок, для которого указана стоимость аренды
				</price>
				<rent>Дополнительные параметры, относящиеся к аренде</rent>
				<owner>
					Владелец объекта: ФИО, контактная информация, группа и компания (если владелец - агент и принадлежит к какой-то компании). Если объект рекламируется частным лицом - тут информация о нём.
				</owner>
				<comment>Комментарий к сделке в произвольной текстовой форме</comment>
				<mortgage>Данные об ипотеке, субсидиях</mortgage>
				<estate>Параметры объекта недвижимости</estate>
				<meta>Метаинформация: когда и откуда импортирован, сопутствующие файлы, etc</meta>
			</order>
			<order>...</order>
			<order>...</order>
			...
		</orders>
	</root>

Элементы внутри order могут содержать текст, числа, логические значения и словарные значения. Общее правило: если тег отсутствует (а отсутствовать могут практически все теги), значение не известно или не определено. Если тег присутствует, то его значение определено и должно быть валидным.

Если элемент содержит произвольный текст (как title или comment > text), то этот текст заключается внутрь

	<![CDATA[...]]>

Ограничений на длину текста нет нигде.

Если элемент содержит число, то и хорошо. Это может быть целое или вещественное число. Однако есть нюанс. Вообще говоря, order может описывать не один объект недвижимости, а сразу диапазон похожих объектов. Это редкая ситуация, которая при импорте и экспорте вряд ли будет встречаться. Тем не менее, у каждого элемента, который должен содержать числовое значения есть необязательный атрибут max, который содержит верхнюю границу значения для описываемого диапазона. Число, заключённое внутри тега, описывает, соответственно, нижнюю границу. Например,

	<price><full max="2000000">1500000</full></price>

описывает цену от полутора до двух миллионов.

Если элемент содержит логическое значение, то оно записывается как true или false. Означает наличие или отсутствие какого-то признака.

Если элемент содержит словарное значение, то у него должен присутствовать числовой атрибут id, указывающий на id записи в соответствующем словаре. Значение, заключённое внутри тега, описывает строковое представление этой записи. Каждый конкретный элемент, который должен содержать словарное значение, связан с конкретным словарём. В примере у таких элементов присутствует необязательный атрибут dic, в котором указано название словаря. В реальных файлах этот атрибут может и должен быть опущен. Пример:

	<type dic="order_type" id="2">покупка</type>

Здесь значение "покупка" имеет id=2 в словаре order_type.

При импорте данных может сложиться такая ситуация, что в словаре внешнего источника есть какое-то значение, которое отсутствует в нашем словаре. Например, в чужом формате встречается тип сделки "субаренда", которого нет у нас. Такие конфликты случаются достаточно часто. В подобной ситуации, если невозможно провести какого-то однозначного соответствия между значениями и нельзя расширить наш собственный словарь, можно использовать элемент без указания атрибута id. Таким образом, информация при импорте не потеряется, просто будет храниться в немного менее структурированном виде. Пример:

	<type>субаренда</type>.

Список обязательных полей
-------------------------

Обязательных полей с точки зрения схемы валидации крайне немного, так как формат призван быть гибким. Однако и при импорте, и при экспорте нужно стараться указать как можно больше значений во имя богатства описания. Обязательные поля следующие:

 - type
 - estate > type
 - estate > object
 - estate > commerce > purpose

Надо отметить, что только элемент type является безусловно обязательным. Скажем, тип объекта недвижимости нужно указывать, только если есть тег estate.

Списки строковых и словарных полей
----------------------------------

Ниже приведены списки строковых и словарных полей. Они более или менее актуальны, однако первичный источник информации - файл с примером.

Строковые поля:

 - title
 - comment > text
 - comment > html
 - comment > short
 - owner > agent > name
 - owner > agent > phone
 - owner > agnet > email
 - estate > location > string
 - estate > desc > text
 - estate > desc > html
 - estate > desc > short
 - estate > house > size
 - estate > house > build_date
 - estate > house > elite > name
 - estate > house > elite > class
 - estate > house > elite > settlement
 - estate > flat > space > desc
 - estate > transport > car > highway
 - meta > url
 - meta > link
 - meta > attachments > attachment > type
 - meta > attachments > attachment > url
 - meta > attachments > attachment > title
 - meta > attachments > attachment > mime

Словарные поля (в скобках указано название словаря)

 - type (order_type)
 - price > rent_period (rent_period)
 - owner > agent (employee)
 - owner > group (group)
 - owner > company (company)
 - mortgage > term (additional_terms)
 - mortgage > bank (company)
 - mortgage > safebox > bank (company)
 - estate > location > country (country)
 - estate > location > region (fias)
 - estate > location > area (fias)
 - estate > location > city (fias)
 - estate > location > district (district)
 - estate > location > street (fias)
 - estate > location > house (fias)
 - estate > type (estate_type)
 - estate > object (object_type)
 - estate > house > type (house_type)
 - estate > house > series (house_series)
 - estate > house > ready_status (ready_status)
 - estate > flat > flat_style (flat_style)
 - estate > flat > state (object_state)
 - estate > storey > storey_type (storey_type)
 - estate > commerce > purpose (purpose_type)
 - estate > commerce > security_type (security_type)
 - estate > commerce > state (object_state)
 - estate > transport > railway > station (railway_station)
 - estate > transport > railway > type (after_transport)
 - estate > transport > metro > station (metro_station)
 - estate > transport > metro > type (after_transport)
 - estate > facilities > gas (gas_type)
 - estate > facilities > bath (bath_type)
 - estate > facilities > sewer (sewer_type)
 - estate > facilities > water (water_type)
 - estate > facilities > heating (heating_type)
 - estate > facilities > decoration (decoration_type)
 - estate > facilities > balcony (balcony_type)
 - estate > facilities > windows (windows_type)
 - estate > facilities > view (view_type)
 - estate > facilities > floor (floor_type)
 - estate > facilities > parking (parking_type)
 - meta > source (infosource)

Конкретные теги
---------------

Дальше приведена информация по конкретным тегам элемента order и значениям, которые они могут содержать.

### type

Тип сделки. Словарь order_type. Единственный обязательный тег.

### price

Цена. Имеет вложенные теги full (полная стоимость), per_meter (стоимость за метр квадратный) и commission (комиссия агента и клиента). Все цены всегда в рублях, если не указан атрибут currency у вложенных тегов full и per_meter. Комиссия - в процентах, если не указана, то клиентская - 100%, агентская - 50%. Если тип сделки - продажа, то цены - это цены продажи. Если тип - сдача в аренду, то цены - цены аренды. Цена аренды указывается за месяц, если значение тега rent > short - false (долгосрочная аренда) и за день, если значение тега rent > short - true (краткосрочная аренда).

Если не указан атрибут currency, то цена считается указанной в рублях. Если указан - то в той валюте, в которой указано. Пока что разрешены следующие валюты: RUR (рубли), EUR (евро), USD (доллары США). В будущем, возможно, добавятся другие.

Если присутствует тег price, но отсутствуют вложенные теги full и/или per_meter, то считается, что цена договорная.

### group

Группировка объектов, позволяющая вкладывать объекты друг в друга. Например, можно описать жилой комплекс, состоящий из множества квартир. Обязательный вложенный тег - type, задающий тип группы объектов (словарь group_type). Необязательный вложенный тег - members - члены группы. Членами группы должны быть полноценные элементы order (которые, в свою очередь, могут группировать другие элементы, если это необходимо).

### rent

Арендная специфика. Вложенный тег term указывает минимальный срок, на который может сдаваться объект. Единица измерения - месяц. Вложенный тег pledge содержит логическое значение, которое истинно тогда, когда за объект необходимо вносить залог. Логический вложенный тег short указывает на то, что аренда - краткосрочная (посуточная). Вложенные теги tenants и neighbors описывают количество съёмщиков и соседей.

### owner

Описывается персона (и опционально - компания и подразделение внутри компании), которая ответственна за сделку. Если речь идёт об объекте на экспорт, то тут хранится информация об агенте, который занимается сделкой. В случае импорта тут может быть информация о частнике-владельце или об агентствах-конкурентах.

Вложенный тег agent описывает физ.лицо (агента или частника). У этого тега может быть атрибут id, указывающий на словарь employee. В тег вложены теги name, phone и email, определяющие, соответственно, ФИО, телефон и email человека. Если телефонов или адресов электронной почты несколько, то в тегах phone и email указываются основные. Дополнительные перечисляются в тегах phone и email, вложенных в тег additional (owner > agent > additional). Атрибут id этих тегов (как основных, так и вложенных в additional) указывает на словарь contact_ownership и позволяет определить, является ли контакт домашним, рабочим, мобильным и т.д.

Вложенные словарные теги company и group обозначают, какой компании и какому подразделению внутри компании принадлежит объект. Тег url позволяет указать URL сайта компании.

При импорте может быть ситуация, когда известна только компания и телефон. В этом случае, название компании указывается в теге company (желательно найти перед этим компанию в словаре), телефон - в теге agent > phone.

### comment

Необязательный текстовый комментарий к сделке. Хранится во вложенном обязательном теге text. Есть дополнительные вложенные теги html и short для, соответственно, HTML-версии и краткого описания.

### mortgage

Всё, что относится к ипотеке. Если тег имеется, значит, ипотека возможна. Если тег отсутствует, ипотека невозможна. Необязательные вложенные теги:

 - term - словарное значение, определяющее дополнительное условие сделки (словарь additional_terms). Один тег term - одно условие. Тегов (и условий) может быть несколько.
 - bank - словарное значение, определяющее банк, дающий ипотеку
 - bank_approval - логическое значение, указывающее на то, одобрил ли банк выдачу кредита
 - summ - сумма кредита
 - date - дата выдачи кредита
 - borrower - ФИО заёмщика
 - contract - номер кредитного договора

### safebox

Данные, относящиеся к банковской ячейке. Необязательный вложенный тег bank - словарное значение, определяющее банк, в котором арендована ячейка. Необязательный вложенный тег date определяет дату выемки денег из ячейки.

### estate

Описание объекта недвижимости. Очень большой, но необязательный тег. Для импорта и экспорта - основной.

 - location - местонахождение объекта. Строковый адрес, точный адрес с указанием кодов ФИАС, GPS-координаты. Всё необязательно.
     - string - произвольная строка, описывающая местонахождение объекта. Обычно является плодом фантазии агента и одновременно его возможностью повлиять на продажи.
     - zip - почтовый индекс
     - country - страна, словарное значение
     - region - регион, словарное значение (ФИАС)
     - area - район региона, словарное значение (ФИАС)
     - city - город, словарное значение (ФИАС)
     - district - район города, словарное значение. Особенность в том, что районы Петербурга и Москвы не классифицируются в базе данных ФИАС (бывший КЛАДР). Поэтому данное поле имеет смысл только для этих двух городов. Районы, скажем, Ленобласти - это уже районы региона, и в ФИАС имеются.
     - street - улица, словарное значение (ФИАС)
     - house - номер дома, опционально - словарное значение (ФИАС)
     - gps - GPS-координаты
         - lat - широта, вещественное число
         - lng - долгота, вещественное число
 - type - тип недвижимости, словарное значение. Редкий обязательный тег.
 - object - тип объекта недвижимости, словарное значение. Редкий обязательный тег.
 - desc - текстовое описание объекта. Те же правила и вложенные теги, что и у тега comment в сделке
 - house - параметры дома. Если тег отсутствует, считается, что отсутствует дом (это имеет смысл для земельных участков и ещё чего-нибудь). Все вложенные теги необязательны.
     - type - тип дома
     - series - серия дома
     - size - размер дома в метрах, синтаксис WIDTHxHEIGHT, например, "10x12"
     - build\_date - точная дата постройки
     - build\_quarter - квартал постройки
     - build\_year - год постройки
     - ready\_status - словарное значение, определяющее, в каком состоянии дом (имеет смысл для новостроек)
     - repair\_year - год капремонта
     - elite - параметры элитной недвижимости
         - name - индивидуальное название дома, например, "Бодрый Лось"
         - penthouse - логическое значение - а не пентхауз ли это
         - townhouse - логическое значение на тему таунхауса
         - class - строка, определяющая класс недвижимости
         - settlement - название посёлка, в котором находится дом
 - flat - параметры жилого помещения. Если тег отсутствует, считается, что такое помещение отсутствует. Все вложенные теги необязательны.
     - space - площади. Все - вещественные числа в квадратных метрах.
         - total - общая площадь
         - living - жилая площадь
         - kitchen - площадь кухни
         - hall - площадь холла
         - desc - строковое поле, описывающее площади комнат в примерно таком виде: "10+12+20"
     - height - вещественное число, высота полотков в метрах
	 - levels - количество уровней (внутренних этажей) в квартире, по умолчанию - 1
	 - rooms - комнатность. Все вложенные теги необязательны.
	     - total - полное количество комнат
	     - actual - продающееся или сдающееся количество комнат. Имеет смысл только тогда, когда объект продаётся или сдаётся не целиком.
	 - style - словарное значение, описывающее народный тип квартиры (распашонка, смежная, etc)
	 - storey - этаж. Все вложенные теги необязательны.
	     - total - полное количество этажей в здании (на данной лестнице)
	     - actual - этаж, на котором находится объект
	     - type - тип этажа (мансарда, подвал, бельэтаж)
	 - state - состояние жилого помещения, словарное значение
	 - elite - параметры элитного жилья
	     - bedrooms - количество спален
	     - bathrooms - количество ванных комнат и санузлов
 - plot - параметры земельного участка. Если тег отсутствует, считается, что объект летает в воздухе. В том смысле, что о земле речь не идёт. Все вложенные теги необязательны
     - purpose - целевое назначение земельного участка.
     - space - площадь участка в квадратных метрах
     - buildings - дополнительные строения на участке
         - sauna - сауна или баня, логическое значение
         - shed - сарай или подсобка, логическое значение
         - garage - гараж, логическое значение
 - commerce - параметры коммерческой недвижимости
     - purpose - целевое назначение коммерческого объекта. Один из немногих обязательных тегов.
     - space - площади в квадратных метрах
         - total - полная площадь объекта
         - business - площадь торговых помещений (если релевантно)
     - subrent - логическое значение, возможна ли субаренда
     - storey - этажи
	     - total - полное количество этажей в здании (на данной лестнице)
	     - actual - этаж, на котором находится объект. Если объект находится на нескольких этажах, а с коммерцией такое бывает, стандартным образом указывается диапазон с помощью атрибута max.
	 - state - состояние коммерческого помещения, словарное значение
	 - max\_energy - число, определяющее полную электрическую мощность, доступную на объекте
	 - phonelines - количество телефонных линий на объекте. Необязательный логический атрибут add указывает, возможно ли увеличение этого количества.
 - transport - описание того, как добираться до объекта
     - car - автомобиль
         - distance - расстояние (от какого-то эталонного места, от Питера, если речь о Ленобласти)
         - time - время, за которое обычно проезжается это расстояние
         - highway - шоссе. Тегов highway может быть произвольное количество, они перечисляют те шоссе, с помощью которых можно добраться до объекта
     - railway - электрички
         - station - словарное значение, железнодорожная станция
         - distance - расстояние, проезжаемое на электричке
         - time - время, за которое электричка проезжает это расстояние
         - type - как добираться до объекта после электрички (пешком или транспортом)
     - metro - метро
         - station - словарное значение, станция метро
         - distance - расстояние от объекта до станции
         - time - время, за которое можно добраться от объекта до станции
         - type - как добираться до объекта после метро (пешком или транспортом)
 - facilities - всяческие прочие параметры объекта
     - bath - словарь, тип ванной комнаты
     - sewer - словарь, тип санузла
     - heating - словарь, тип отопления
     - decoration - словарь, тип отделки
     - balcony - словарь, тип балкона
     - windows - словарь, тип окон
     - view - словарь, вид из окна
     - floor - словарь, тип напольного покрытия
     - gas - логическое значение, газоснабжение
     - water - логическое значение, водоснабжение
     - parking - логическое значение, парковка
	 - electricity - логическое значение, электричество
	 - concierge - логическое значение, консьерж
	 - guard - логическое значение, охрана
	 - fridge - логическое значение, холодильник
	 - washmachine - логическое значение, стиральная машина
	 - tv - логическое значение, телевизор
	 - aircon - логическое значение, кондиционер
	 - garbage\_chute - логическое значение, мусоропровод
	 - internet - логическое значение, интернет
	 - phone - логическое значение, телефон
	 - elevator - логическое значение, лифт
	 - furniture - логическое значение, мебель в жилых или коммерческих помещениях
	 - kitchen_furniture - логическое значение, мебель на кухне
	 - pets_allowed - логическое значение, разрешены ли животные
	 - children_allowed - логическое значение, разрешены ли дети

### meta

Метаинформация об объекте.

Для импортированных объектов указывается источник (source), дата и время импорта (imported}, URL источника данных (url), URL конкретного объекта на сайте-источнике (link), ID объекта в источнике (extid), степень выделенности объекта в издании (highlight), любые дополнительные структурированные данные (extra). Все теги необязательны.

Значение в теге highlight - это число, которое равно 0, если объявление не выделено, и больше нуля, если объявление выделено. Чем больше это число, тем выделеннее объявление. Каждое издание позволяет выделять объявления по-своему, некоторые не позволяют вообще. Как именно преобразовывается выделение в число (и обратно) - остаётся на совести конвертера. Пример для БН: 0 - обычная строка, 1 - жирная строка, 2 - золотая строка. Пример для БКН: 0 - обычная строка, 1 - жирная строка. Пример для старого формата ИРР: 0 - обычное объявление, 1 - треугольник. Пример для ГдеЭтогоДома: 0 - обычное объявление, 1 - выделить рамкой (is_border=true), 2 - выделить цветом (is_premium=true).

Кроме того, с объектом могут идти какие-то связанные файлы, например, фотографии, планировки, видеообзоры, etc. Для этого служит коллекция attachments, каждый элемент attachment которой описывает отдельный файл.

 - extra - любые дополнительные структурированные данные в JSON (внутри CDATA). Они сохраняются в базе и, возможно, отображаются в карточке объекта, однако по ним невозможен поиск.
 - url - URL файла, единственный обязательный элемент
 - type - тип файла в виде строки, например, image, planning, video, document. Имеет смысл, в основном, для экспорта. Впрочем, если при импорте удаётся отделить фотографии объекта от планировок, то для фотографий тип должен быть image, для планировок - planning.
 - title - строка, человекопонятное название файла
 - hash - хэши, уникальные для данного файла
     - md5
     - sha1
     - sha256
 - mime - MIME-тип файла
 - size - размер файла в байтах
 - width - ширина изображения или видео
 - height - высота изображения или видео
 - length - длина видео в секундах








