# Обычные формы

Процедура КнопкаВыполнитьНажатие(Кнопка)
	
	Если РежимВыгрузки Тогда
		
		Адрес = ВернутьДанныеВоВременномХранилище();
		
		Если Адрес = Неопределено Тогда 
			Возврат;
		КонецЕсли;
		
		ПараметрыСохранения = ФайловаяСистемаКлиент.ПараметрыСохраненияФайла();
		ПараметрыСохранения.Диалог.Заголовок = НСтр("ru = 'Сохранить данные по цессиям в файл'");
		ПараметрыСохранения.Диалог.Фильтр    = "Файлы (*.json)|*.json";
		
		ФайловаяСистемаКлиент.СохранитьФайл(Неопределено, Адрес,, ПараметрыСохранения);
		
	Иначе
		
		Если НЕ ПроверитьЗаполнение() Тогда 
			Возврат;
		КонецЕсли;	
		
		Если ВалютнаяПроводка И ВалютаПроводки.Пустая() Тогда
			
			ОбщегоНазначения.СообщитьИнформациюПользователю("Не заполнено поле ""Валюта проводки""");
			Возврат;
			
		КонецЕсли;	
		
		ОписаниеОповещения = Новый ОписаниеОповещения("ОбработатьФайлЗавершение", ЭтаФорма);
		
		ПараметрыЗагрузки = ФайловаяСистемаКлиент.ПараметрыЗагрузкиФайла();
		ПараметрыЗагрузки.ИдентификаторФормы = Новый УникальныйИдентификатор;
		ПараметрыЗагрузки.Диалог.Заголовок   = НСтр("ru = 'Выбрать файл к загрузке'");
		ПараметрыЗагрузки.Диалог.Фильтр      = "Файлы (*.json)|*.json";
		
		ФайловаяСистемаКлиент.ЗагрузитьФайл(ОписаниеОповещения, ПараметрыЗагрузки);
		
	КонецЕсли;
	
КонецПроцедуры

Процедура ОбработатьФайлЗавершение(ПомещенныйФайл, ДополнительныеПараметры) Экспорт 

	Если ПомещенныйФайл = Неопределено ИЛИ ПомещенныйФайл.Количество() = 0 Тогда
		
		ОбщегоНазначения.СообщитьИнформациюПользователю("Не выбран файл пользователем!");	
		Возврат;
		
	КонецЕсли;
	
	ВыполнитьЗагрузкуЦессий(ПомещенныйФайл.Хранение);
	
	ОбщегоНазначения.СообщитьИнформациюПользователю("Цессии перенесены успешно!");
	
КонецПроцедуры

Функция ВернутьДанныеВоВременномХранилище() Экспорт 
	
	Если СтрДлина(ИмяФайлаФК) = 0 Тогда
		
		ОбщегоНазначения.СообщитьИнформациюПользователю("Не выбран файл для загрузки, в поле ""Имя файла ФК""!");	
		Возврат Неопределено;
		
	КонецЕсли;
		
	ТабличныйДокумент = Новый ТабличныйДокумент;
	
	Попытка
		ТабличныйДокумент.Прочитать(ИмяФайлаФК);
	Исключение
		
		ОбщегоНазначения.СообщитьИнформациюПользователю("Не получилось прочитать файл, по причине: " + ОписаниеОшибки());
		Возврат Неопределено;
		
	КонецПопытки;
	
	Построитель = Новый ПостроительЗапроса;
	Построитель.ИсточникДанных          = Новый ОписаниеИсточникаДанных(ТабличныйДокумент.Область());
	Построитель.ДобавлениеПредставлений = ТипДобавленияПредставлений.НеДобавлять;
	Построитель.ЗаполнитьНастройки();
	Построитель.Выполнить();
	
	Попытка
		ВнешняяТаблица = Построитель.Результат.Выгрузить();
	Исключение
		//При выводе в таблицу значений через построитель запроса, 
		//названия колонок в источнике данных должны быть первой строкой, 
		//иначе построитель не сможет получить и преобразовать их в колонки таблицы значений и выдаст ошибку.
		ОбщегоНазначения.СообщитьИнформациюПользователю("Нужно удалить в загружаемом файле - первые, пустые строки!");
		Возврат Неопределено;
		
	КонецПопытки;	
	
	Если ВнешняяТаблица.Количество() = 0 Тогда
		
		ОбщегоНазначения.СообщитьИнформациюПользователю("Нет данных для загрузки, обработка прекращена!");
		Возврат Неопределено;
		
	КонецЕсли;
	
	Запрос = Новый Запрос;
	Запрос.УстановитьПараметр("ВнешняяТаблица", ВнешняяТаблица);
	Запрос.Текст = 
		"ВЫБРАТЬ
		|	ТаблицаКонтрактов.НомерКонтракта КАК НомерКонтракта,
		|	ТаблицаКонтрактов.Баланс КАК Баланс,
		|	ТаблицаКонтрактов.Переоценка КАК Переоценка
		|ПОМЕСТИТЬ ВТ_ТаблицаКонтрактов
		|ИЗ
		|	&ВнешняяТаблица КАК ТаблицаКонтрактов
		|
		|ИНДЕКСИРОВАТЬ ПО
		|	НомерКонтракта
		|;
		|
		|////////////////////////////////////////////////////////////////////////////////
		|ВЫБРАТЬ
		|	ФорвардныйКонтракт.Ссылка,
		|	ФорвардныйКонтракт.ВидОперации,
		|	ФорвардныйКонтракт.ДоговорПоНомеруСделки,
		|	ФорвардныйКонтракт.ДатаИсполнения,
		|	ПРЕДСТАВЛЕНИЕ(ФорвардныйКонтракт.БазовыйАктив) КАК БазовыйАктив,
		|	ПРЕДСТАВЛЕНИЕ(ФорвардныйКонтракт.БиржаДляРасчетов) КАК БиржаДляРасчетов,
		|	ПРЕДСТАВЛЕНИЕ(ФорвардныйКонтракт.ВалютаДокумента) КАК ВалютаДокумента,
		|	ПРЕДСТАВЛЕНИЕ(ФорвардныйКонтракт.ВалютаИсполнения) КАК ВалютаИсполнения,
		|	ПРЕДСТАВЛЕНИЕ(ФорвардныйКонтракт.ВалютаПремии) КАК ВалютаПремии,
		|	ПРЕДСТАВЛЕНИЕ(ФорвардныйКонтракт.ВалютаРасчетов) КАК ВалютаРасчетов,
		|	ПРЕДСТАВЛЕНИЕ(ФорвардныйКонтракт.КонтрагентВтораяСторона) КАК КонтрагентВтораяСторона,
		|	ПРЕДСТАВЛЕНИЕ(ФорвардныйКонтракт.МестоСовершения) КАК МестоСовершения,
		|	ПРЕДСТАВЛЕНИЕ(ФорвардныйКонтракт.ТипКонтракта) КАК ТипКонтракта,
		|	ВТ_ТаблицаКонтрактов.Баланс,
		|	ВТ_ТаблицаКонтрактов.Переоценка
		|ПОМЕСТИТЬ ВТ_ФК
		|ИЗ
		|	ВТ_ТаблицаКонтрактов КАК ВТ_ТаблицаКонтрактов
		|		ВНУТРЕННЕЕ СОЕДИНЕНИЕ Документ.ФорвардныйКонтракт КАК ФорвардныйКонтракт
		|		ПО ВТ_ТаблицаКонтрактов.НомерКонтракта = ФорвардныйКонтракт.НомерДоговора
		|			И (ФорвардныйКонтракт.Проведен)
		|
		|ИНДЕКСИРОВАТЬ ПО
		|	ФорвардныйКонтракт.ВидОперации
		|;
		|
		|////////////////////////////////////////////////////////////////////////////////
		|ВЫБРАТЬ
		|	ВТ_ФК.Ссылка,
		|	ВТ_ФК.ДоговорПоНомеруСделки,
		|	ВТ_ФК.ДатаИсполнения,
		|	ВТ_ФК.БазовыйАктив,
		|	ВТ_ФК.БиржаДляРасчетов,
		|	ВТ_ФК.ВалютаДокумента,
		|	ВТ_ФК.ВалютаИсполнения,
		|	ВТ_ФК.ВалютаПремии,
		|	ВТ_ФК.ВалютаРасчетов,
		|	ВТ_ФК.КонтрагентВтораяСторона,
		|	ВТ_ФК.МестоСовершения,
		|	ВТ_ФК.ТипКонтракта,
		|	ВТ_ФК.Баланс,
		|	ВТ_ФК.Переоценка
		|ПОМЕСТИТЬ ВТ_Заключения
		|ИЗ
		|	ВТ_ФК КАК ВТ_ФК
		|ГДЕ
		|	ВТ_ФК.ВидОперации = ЗНАЧЕНИЕ(Перечисление.ВидыФорвардныхОпераций.Заключение)
		|
		|ИНДЕКСИРОВАТЬ ПО
		|	ВТ_ФК.ДоговорПоНомеруСделки
		|;
		|
		|////////////////////////////////////////////////////////////////////////////////
		|ВЫБРАТЬ
		|	МАКСИМУМ(ВложенныйЗапрос.Ссылка) КАК Ссылка,
		|	ВложенныйЗапрос.ДоговорПоНомеруСделки КАК ДоговорПоНомеруСделки
		|ПОМЕСТИТЬ ВТ_ДопСобытия
		|ИЗ
		|	(ВЫБРАТЬ
		|		ВТ_ФК.Ссылка КАК Ссылка,
		|		ВТ_ФК.ДоговорПоНомеруСделки КАК ДоговорПоНомеруСделки
		|	ИЗ
		|		ВТ_ФК КАК ВТ_ФК
		|	ГДЕ
		|		ВТ_ФК.ВидОперации = ЗНАЧЕНИЕ(Перечисление.ВидыФорвардныхОпераций.КорпоративноеСобытие)
		|	
		|	ОБЪЕДИНИТЬ ВСЕ
		|	
		|	ВЫБРАТЬ
		|		ВТ_ФК.Ссылка,
		|		ВТ_ФК.ДоговорПоНомеруСделки
		|	ИЗ
		|		ВТ_ФК КАК ВТ_ФК
		|	ГДЕ
		|		ВТ_ФК.ВидОперации = ЗНАЧЕНИЕ(Перечисление.ВидыФорвардныхОпераций.ДопСоглашение)
		|	
		|	ОБЪЕДИНИТЬ ВСЕ
		|	
		|	ВЫБРАТЬ
		|		ВТ_ФК.Ссылка,
		|		ВТ_ФК.ДоговорПоНомеруСделки
		|	ИЗ
		|		ВТ_ФК КАК ВТ_ФК
		|	ГДЕ
		|		ВТ_ФК.ВидОперации = ЗНАЧЕНИЕ(Перечисление.ВидыФорвардныхОпераций.Пролонгация)) КАК ВложенныйЗапрос
		|
		|СГРУППИРОВАТЬ ПО
		|	ВложенныйЗапрос.ДоговорПоНомеруСделки
		|
		|ИНДЕКСИРОВАТЬ ПО
		|	ДоговорПоНомеруСделки
		|;
		|
		|////////////////////////////////////////////////////////////////////////////////
		|ВЫБРАТЬ
		|	ВТ_ФК.ДоговорПоНомеруСделки,
		|	ВТ_ФК.ДатаИсполнения
		|ПОМЕСТИТЬ ВТ_Расторжения
		|ИЗ
		|	ВТ_ФК КАК ВТ_ФК
		|ГДЕ
		|	ВТ_ФК.ВидОперации = ЗНАЧЕНИЕ(Перечисление.ВидыФорвардныхОпераций.Расторжение)
		|
		|ИНДЕКСИРОВАТЬ ПО
		|	ВТ_ФК.ДоговорПоНомеруСделки
		|;
		|
		|////////////////////////////////////////////////////////////////////////////////
		|ВЫБРАТЬ
		|	ВТ_Заключения.Ссылка КАК ДокументЗаключения,
		|	ЕСТЬNULL(ВТ_ДопСобытия.Ссылка, ЗНАЧЕНИЕ(Документ.ФорвардныйКонтракт.ПустаяСсылка)) КАК ДокументДопСобытия,
		|	ЕСТЬNULL(ВТ_Расторжения.ДатаИсполнения, ВТ_Заключения.ДатаИсполнения) КАК ДатаИсполнения,
		|	ВТ_Заключения.БазовыйАктив,
		|	ВТ_Заключения.БиржаДляРасчетов,
		|	ВТ_Заключения.ВалютаДокумента,
		|	ВТ_Заключения.ВалютаИсполнения,
		|	ВТ_Заключения.ВалютаПремии,
		|	ВТ_Заключения.ВалютаРасчетов,
		|	ВТ_Заключения.КонтрагентВтораяСторона,
		|	ВТ_Заключения.МестоСовершения,
		|	ВТ_Заключения.ТипКонтракта,
		|	ВТ_Заключения.Баланс,
		|	ВТ_Заключения.Переоценка
		|ПОМЕСТИТЬ ВТ_Цессии
		|ИЗ
		|	ВТ_Заключения КАК ВТ_Заключения
		|		ЛЕВОЕ СОЕДИНЕНИЕ ВТ_ДопСобытия КАК ВТ_ДопСобытия
		|		ПО ВТ_Заключения.ДоговорПоНомеруСделки = ВТ_ДопСобытия.ДоговорПоНомеруСделки
		|		ЛЕВОЕ СОЕДИНЕНИЕ ВТ_Расторжения КАК ВТ_Расторжения
		|		ПО ВТ_Заключения.ДоговорПоНомеруСделки = ВТ_Расторжения.ДоговорПоНомеруСделки
		|			И ВТ_Заключения.ДатаИсполнения < ВТ_Расторжения.ДатаИсполнения
		|
		|ИНДЕКСИРОВАТЬ ПО
		|	ДокументЗаключения,
		|	ДокументДопСобытия
		|;
		|
		|////////////////////////////////////////////////////////////////////////////////
		|ВЫБРАТЬ
		|	ОбъектыИмпортаИзБО.УИд КАК ИдентификаторИмпорта,
		|	ВЫРАЗИТЬ(ОбъектыИмпортаИзБО.СсылкаНаИмпортированныйОбъект КАК Документ.ФорвардныйКонтракт) КАК СсылкаНаИмпортированныйОбъект
		|ПОМЕСТИТЬ ВТ_ОбъектыИмпорта
		|ИЗ
		|	РегистрСведений.ОбъектыИмпортаИзБО КАК ОбъектыИмпортаИзБО
		|ГДЕ
		|	ОбъектыИмпортаИзБО.СсылкаНаИмпортированныйОбъект ССЫЛКА Документ.ФорвардныйКонтракт
		|	И ОбъектыИмпортаИзБО.СсылкаНаИмпортированныйОбъект <> ЗНАЧЕНИЕ(Документ.ФорвардныйКонтракт.ПустаяСсылка)
		|
		|ИНДЕКСИРОВАТЬ ПО
		|	СсылкаНаИмпортированныйОбъект
		|;
		|
		|////////////////////////////////////////////////////////////////////////////////
		|ВЫБРАТЬ
		|	ВТ_Цессии.ДокументЗаключения,
		|	ЕСТЬNULL(ОбъектыИмпортаЗаключения.ИдентификаторИмпорта, """") КАК ИдентификаторЗаключения,
		|	ЕСТЬNULL(ОбъектыИмпортаДопСобытия.ИдентификаторИмпорта, """") КАК ИдентификаторДопСобытия,
		|	ВТ_Цессии.ДатаИсполнения,
		|	ВТ_Цессии.БазовыйАктив,
		|	ВТ_Цессии.БиржаДляРасчетов,
		|	ВТ_Цессии.ВалютаДокумента,
		|	ВТ_Цессии.ВалютаИсполнения,
		|	ВТ_Цессии.ВалютаПремии,
		|	ВТ_Цессии.ВалютаРасчетов,
		|	ВТ_Цессии.КонтрагентВтораяСторона,
		|	ВТ_Цессии.МестоСовершения,
		|	ВТ_Цессии.ТипКонтракта,
		|	ВТ_Цессии.Баланс,
		|	ВТ_Цессии.Переоценка
		|ИЗ
		|	ВТ_Цессии КАК ВТ_Цессии
		|		ЛЕВОЕ СОЕДИНЕНИЕ ВТ_ОбъектыИмпорта КАК ОбъектыИмпортаЗаключения
		|		ПО ВТ_Цессии.ДокументЗаключения = ОбъектыИмпортаЗаключения.СсылкаНаИмпортированныйОбъект
		|		ЛЕВОЕ СОЕДИНЕНИЕ ВТ_ОбъектыИмпорта КАК ОбъектыИмпортаДопСобытия
		|		ПО ВТ_Цессии.ДокументДопСобытия = ОбъектыИмпортаДопСобытия.СсылкаНаИмпортированныйОбъект";
	
	РезультатЗапроса = Запрос.Выполнить();
	
	Если РезультатЗапроса.Пустой() Тогда
		
		ОбщегоНазначения.СообщитьИнформациюПользователю("Нет данных к выгрузке!");
		Возврат Неопределено;
		
	КонецЕсли;	

	ИмяВременногоФайла = ПолучитьИмяВременногоФайла("json");
	
	ПараметрыЗаписиФайла = Новый ПараметрыЗаписиJSON(, Символы.Таб);
	
	ЗаписьJSON = Новый ЗаписьJSON;
	ЗаписьJSON.ОткрытьФайл(ИмяВременногоФайла,,, ПараметрыЗаписиФайла);
	ЗаписьJSON.ЗаписатьНачалоОбъекта();
	ЗаписьJSON.ЗаписатьИмяСвойства("Цессии");
	ЗаписьJSON.ЗаписатьНачалоМассива();
	
	ВыборкаРезультата = РезультатЗапроса.Выбрать();
	
	Пока ВыборкаРезультата.Следующий() Цикл 
		
		ЗаписьJSON.ЗаписатьНачалоОбъекта();
		
		ДокументОбъект = ВыборкаРезультата.ДокументЗаключения.ПолучитьОбъект();
		
		ЗаписьJSON.ЗаписатьИмяСвойства("ДокументЗаключения");
		СериализаторXDTO.ЗаписатьJSON(ЗаписьJSON, ДокументОбъект);
		
		ДополнительныеСвойства = Новый Структура("ИдентификаторЗаключения, ИдентификаторДопСобытия, ДатаИсполнения, БазовыйАктив, 
			|БиржаДляРасчетов, ВалютаДокумента, ВалютаИсполнения, ВалютаПремии, ВалютаРасчетов, КонтрагентВтораяСторона, МестоСовершения,
			|ТипКонтракта, Баланс, Переоценка");
		
		ЗаполнитьЗначенияСвойств(ДополнительныеСвойства, ВыборкаРезультата);
		
		Если ДокументОбъект.ПараметрыКонтракта.Количество() > 0 Тогда 
			
			Запрос = Новый Запрос;
			Запрос.Текст = 
				"ВЫБРАТЬ
				|	Таблица.НомерСтроки,
				|	Таблица.Актив
				|ПОМЕСТИТЬ ВТ_ТаблицаАктивов
				|ИЗ
				|	&ВнешняяТаблица КАК Таблица
				|;
				|
				|////////////////////////////////////////////////////////////////////////////////
				|ВЫБРАТЬ
				|	ТаблицаАктивов.НомерСтроки КАК НомерСтроки,
				|	ЕСТЬNULL(ОбъектыИмпортаИзБО.УИд, """") КАК УИдАктива
				|ИЗ
				|	ВТ_ТаблицаАктивов КАК ТаблицаАктивов
				|		ЛЕВОЕ СОЕДИНЕНИЕ РегистрСведений.ОбъектыИмпортаИзБО КАК ОбъектыИмпортаИзБО
				|		ПО (ТаблицаАктивов.Актив = (ВЫРАЗИТЬ(ОбъектыИмпортаИзБО.СсылкаНаИмпортированныйОбъект КАК Справочник.СпецификацииСрочныхКонтрактов)))
				|
				|УПОРЯДОЧИТЬ ПО
				|	НомерСтроки";
			
			Запрос.УстановитьПараметр("ВнешняяТаблица", ДокументОбъект.ПараметрыКонтракта.Выгрузить(, "НомерСтроки, Актив"));
			ДополнительныеСвойства.Вставить("АктивыКонтракта", Запрос.Выполнить().Выгрузить().ВыгрузитьКолонку("УИдАктива"));
			
		КонецЕсли;	
		
		ЗаписьJSON.ЗаписатьИмяСвойства("ДополнительныеСвойства"); 
		ЗаписатьJSON(ЗаписьJSON, ДополнительныеСвойства);
		
		ЗаписьJSON.ЗаписатьКонецОбъекта();
		
	КонецЦикла;	
	 
	ЗаписьJSON.ЗаписатьКонецМассива();
	ЗаписьJSON.ЗаписатьКонецОбъекта();
	
	ЗаписьJSON.Закрыть();
	
	АдресВоВременномХранилище = ПоместитьВоВременноеХранилище(Новый ДвоичныеДанные(ИмяВременногоФайла), Новый УникальныйИдентификатор);
	
	ФайловаяСистема.УдалитьВременныйФайл(ИмяВременногоФайла);
	
	Возврат АдресВоВременномХранилище;

КонецФункции

Процедура ВыполнитьЗагрузкуЦессий(АдресВоВременномХранилище) Экспорт
	
	ИмяВременногоФайла = ПолучитьИмяВременногоФайла("json");
	
	ДвоичныеДанные = ПолучитьИзВременногоХранилища(АдресВоВременномХранилище);
	ДвоичныеДанные.Записать(ИмяВременногоФайла);
	
	ЧтениеJSON = Новый ЧтениеJSON;
	ЧтениеJSON.ОткрытьФайл(ИмяВременногоФайла);
	
	ДанныеПоЦессиям = Новый Массив;
	
	Пока ЧтениеJSON.Прочитать() Цикл
		
		ТипJSON = ЧтениеJSON.ТипТекущегоЗначения;
		
		Если ТипJSON = ТипЗначенияJSON.ИмяСвойства Тогда 
			
			Если ЧтениеJSON.ТекущееЗначение = "ДокументЗаключения" Тогда 
				
				ДокументОбъект = СериализаторXDTO.ПрочитатьJSON(ЧтениеJSON, Тип("ДокументОбъект.ФорвардныйКонтракт"));
				
				ДанныеЦессии = Новый Структура("ДокументЗаключения, ДополнительныеСвойства");
				ДанныеЦессии.ДокументЗаключения = ДокументОбъект;
				
			ИначеЕсли ЧтениеJSON.ТекущееЗначение = "ДополнительныеСвойства" Тогда
				
				ДанныеЦессии.ДополнительныеСвойства = ПрочитатьJSON(ЧтениеJSON,, "ДатаИсполнения", ФорматДатыJSON.ISO);
				ДанныеПоЦессиям.Добавить(ДанныеЦессии);
				
			КонецЕсли;	
			
		КонецЕсли;	
		
	КонецЦикла;	
	
	ЧтениеJSON.Закрыть();
	
	ФайловаяСистема.УдалитьВременныйФайл(ИмяВременногоФайла);
	
	мВалюта = Константы.ВалютаРегламентированногоУчета.Получить();

	НомерИтерации = 1;
	КоличествоЦессий = ДанныеПоЦессиям.Количество();
	
	//Исключаемы поля для заполнения нового документа. Какие-то поля пропущены за ненадобностью, остальные заполняются по алгоритму
	ИсключаяСвойства = "БазовыйАктив, БанковскийСчет, БиржаДляРасчетов, Брокер, ВалютаДокумента, ВалютаИсполнения,
			|ВалютаПремии, ВалютаРасторжения, ВалютаРасчетов, ВерсияДанных, Дата, ДатаИсполнения, ДатаКорпоративногоСобытия,
			|ДатаОграниченияПрава, ДатаОплаты, ДатаПредьявленияПрава, Движения, ДоговорБрокер, ДоговорИсполнения, ДоговорКонтрагента, 
			|ДоговорНеттинг, ДоговорПоНомеруСделки, ДокументОснование, Комментарий, Контрагент, КонтрагентВтораяСторона, КонтрагентНеттинг,
			|МестоСовершения, Номер, ОбменДанными, Организация, Ответственный, ПринадлежностьПоследовательностям, Проведен,
			|СписатьСБрокерскогоСчета, ТипКонтракта, ЭтотОбъект, мВалютаРегламентированногоУчета";
	
	Для каждого КоллекцияДанных из ДанныеПоЦессиям Цикл
		
		ПроцентВыполнения = 100 * НомерИтерации / КоличествоЦессий;
		
		Состояние("Создаются цессии " + Формат(ПроцентВыполнения, "ЧЦ=3; ЧДЦ=0") + " %");
		
		НовыйДокумент = Документы.ФорвардныйКонтракт.СоздатьДокумент();
		
		ЗаполнитьЗначенияСвойств(НовыйДокумент, КоллекцияДанных.ДокументЗаключения,, ИсключаяСвойства);
		
		НовыйДокумент.БазовыйАктив             = ?(ЗначениеЗаполнено(КоллекцияДанных.ДополнительныеСвойства.БазовыйАктив), Справочники.СпецификацииСрочныхКонтрактов.НайтиПоНаименованию(КоллекцияДанных.ДополнительныеСвойства.БазовыйАктив), Справочники.СпецификацииСрочныхКонтрактов.ПустаяСсылка());
		НовыйДокумент.БиржаДляРасчетов         = Справочники._рарБиржи.НайтиПоНаименованию(КоллекцияДанных.ДополнительныеСвойства.БиржаДляРасчетов);
		НовыйДокумент.ВалютаДокумента          = ?(ЗначениеЗаполнено(КоллекцияДанных.ДополнительныеСвойства.ВалютаДокумента), Справочники.Валюты.НайтиПоНаименованию(КоллекцияДанных.ДополнительныеСвойства.ВалютаДокумента), Справочники.Валюты.ПустаяСсылка());
		НовыйДокумент.ВалютаИсполнения         = ?(ЗначениеЗаполнено(КоллекцияДанных.ДополнительныеСвойства.ВалютаИсполнения), Справочники.Валюты.НайтиПоНаименованию(КоллекцияДанных.ДополнительныеСвойства.ВалютаИсполнения), Справочники.Валюты.ПустаяСсылка());
		НовыйДокумент.ВалютаПремии             = ?(ЗначениеЗаполнено(КоллекцияДанных.ДополнительныеСвойства.ВалютаПремии), Справочники.Валюты.НайтиПоНаименованию(КоллекцияДанных.ДополнительныеСвойства.ВалютаПремии), Справочники.Валюты.ПустаяСсылка());
		НовыйДокумент.ВалютаРасчетов           = ?(ЗначениеЗаполнено(КоллекцияДанных.ДополнительныеСвойства.ВалютаРасчетов), Справочники.Валюты.НайтиПоНаименованию(КоллекцияДанных.ДополнительныеСвойства.ВалютаРасчетов), Справочники.Валюты.ПустаяСсылка());
		НовыйДокумент.Дата		               = КонецДня(ДатаЦессии);
		НовыйДокумент.ДатаИсполнения           = КоллекцияДанных.ДополнительныеСвойства.ДатаИсполнения;
		НовыйДокумент.Комментарий              = "Цессия от" + Формат(ДатаЦессии, "ДЛФ=DD");
		НовыйДокумент.Контрагент               = Брокер;
		НовыйДокумент.КонтрагентВтораяСторона  = Справочники.Контрагенты.НайтиПоНаименованию(КоллекцияДанных.ДополнительныеСвойства.КонтрагентВтораяСторона);
		НовыйДокумент.МестоСовершения          = Справочники._рарБиржи.НайтиПоНаименованию(КоллекцияДанных.ДополнительныеСвойства.МестоСовершения);
		НовыйДокумент.Организация              = Организация;
		НовыйДокумент.СписатьСБрокерскогоСчета = Ложь;
		НовыйДокумент.ТипКонтракта             = Справочники.ТипыФорвардныхКонтрактов.НайтиПоНаименованию(КоллекцияДанных.ДополнительныеСвойства.ТипКонтракта);
		
		ПрефиксДоговора = ?(НовыйДокумент.ВидКонтракта = Перечисления.ВидыСрочныхКонтрактов.Опцион, "ок", "фк");
		НомернойДоговор = УправлениеЦБ.СформироватьИмяНомерногоДоговора(НовыйДокумент.НомерДоговора, ПрефиксДоговора);
		
		НовыйДокумент.ДоговорПоНомеруСделки = УправлениеЦБ.ПолучитьДоговор(НомернойДоговор, НовыйДокумент.Организация, НовыйДокумент.Контрагент, ,, Истина);
		НовыйДокумент.ДоговорКонтрагента    = УправлениеЦБ.ПолучитьДоговор("Ген. соглашение (" + НовыйДокумент.ВалютаДокумента + ")", НовыйДокумент.Организация, НовыйДокумент.Контрагент, НовыйДокумент.ВалютаДокумента,, Истина);
		НовыйДокумент.ДоговорИсполнения     = УправлениеЦБ.ПолучитьДоговор("Ген. соглашение (" + НовыйДокумент.ВалютаИсполнения + ")", НовыйДокумент.Организация, НовыйДокумент.Контрагент, НовыйДокумент.ВалютаИсполнения,, Истина);
		
		Если КоллекцияДанных.ДокументЗаключения.ДатыПромежуточныхФиксаций.Количество() > 0 Тогда 
			НовыйДокумент.ДатыПромежуточныхФиксаций.Загрузить(КоллекцияДанных.ДокументЗаключения.ДатыПромежуточныхФиксаций.Выгрузить());
		КонецЕсли;	
		
		Если КоллекцияДанных.ДополнительныеСвойства.Свойство("АктивыКонтракта") Тогда 
			
			ВнешняяТаблица = КоллекцияДанных.ДокументЗаключения.ПараметрыКонтракта.Выгрузить();
			ВнешняяТаблица.Колонки.Добавить("УИдАктива", Новый ОписаниеТипов("Строка",, Новый КвалификаторыСтроки(36)));
			
			Для каждого СтрокаТаблицы Из ВнешняяТаблица Цикл
			
				СтрокаТаблицы.УИдАктива = КоллекцияДанных.ДополнительныеСвойства.АктивыКонтракта[СтрокаТаблицы.НомерСтроки - 1];
			
			КонецЦикла;
			
			Запрос = Новый Запрос;
			Запрос.Текст = 
				"ВЫБРАТЬ
				|	Таблица.НомерСтроки,
				|	Таблица.Параметр,
				|	Таблица.Значение,
				|	Таблица.УИдАктива,
				|	Таблица.КоличествоПоставки,
				|	Таблица.КонечноеЗначение,
				|	Таблица.ЦенаПоставки,
				|	Таблица.ВидСобытия,
				|	Таблица.КоэффициентКонвертации
				|ПОМЕСТИТЬ ВТ_ТаблицаАктивов
				|ИЗ
				|	&ВнешняяТаблица КАК Таблица
				|;
				|
				|////////////////////////////////////////////////////////////////////////////////
				|ВЫБРАТЬ
				|	ТаблицаАктивов.НомерСтроки,
				|	ТаблицаАктивов.Параметр,
				|	ТаблицаАктивов.Значение,
				|	ЕСТЬNULL(ВЫРАЗИТЬ(ОбъектыИмпортаИзБО.СсылкаНаИмпортированныйОбъект КАК Справочник.СпецификацииСрочныхКонтрактов), ЗНАЧЕНИЕ(Справочник.СпецификацииСрочныхКонтрактов.ПустаяСсылка)) КАК Актив,
				|	ТаблицаАктивов.КоличествоПоставки,
				|	ТаблицаАктивов.КонечноеЗначение,
				|	ТаблицаАктивов.ЦенаПоставки,
				|	ТаблицаАктивов.ВидСобытия,
				|	ТаблицаАктивов.КоэффициентКонвертации
				|ИЗ
				|	ВТ_ТаблицаАктивов КАК ТаблицаАктивов
				|		ЛЕВОЕ СОЕДИНЕНИЕ РегистрСведений.ОбъектыИмпортаИзБО КАК ОбъектыИмпортаИзБО
				|		ПО ТаблицаАктивов.УИдАктива = ОбъектыИмпортаИзБО.УИд
				|
				|УПОРЯДОЧИТЬ ПО
				|	ТаблицаАктивов.НомерСтроки";
			
			Запрос.УстановитьПараметр("ВнешняяТаблица", ВнешняяТаблица);
			НовыйДокумент.ПараметрыКонтракта.Загрузить(Запрос.Выполнить().Выгрузить());
			
		КонецЕсли;	
		
		НовыйДокумент.Записать(РежимЗаписиДокумента.Проведение);
		
		НачатьТранзакцию();      
		
		ВалютныйСчет = Ложь;
		
		Если ВалютнаяПроводка И ВалютаПроводки <> мВалюта Тогда
			
			ВалютныйСчет = Истина;
			ВалютаДт     = ВалютаПроводки;
			ВалютаКт     = ВалютаПроводки;

			Запись_мВалюта = РегистрыСведений.КурсыВалют.ПолучитьПоследнее(ДатаЦессии, Новый Структура("Валюта", ВалютаПроводки));
			Курс_мВалюты 	  = ?(ЗначениеЗаполнено(Запись_мВалюта.Курс), Запись_мВалюта.Курс, 1);
			Кратность_мВалюты = ?(ЗначениеЗаполнено(Запись_мВалюта.Кратность), Запись_мВалюта.Кратность, 1);
			
		ИначеЕсли НЕ ВалютнаяПроводка И НовыйДокумент.ВалютаРасчетов <> мВалюта Тогда
			
			ВалютныйСчет = Истина;
			ВалютаДт     = НовыйДокумент.ВалютаРасчетов;
			ВалютаКт     = НовыйДокумент.ВалютаРасчетов;
		
			Запись_мВалюта = РегистрыСведений.КурсыВалют.ПолучитьПоследнее(ДатаЦессии, Новый Структура("Валюта", НовыйДокумент.ВалютаРасчетов));
			Курс_мВалюты 	  = ?(ЗначениеЗаполнено(Запись_мВалюта.Курс), Запись_мВалюта.Курс, 1);
			Кратность_мВалюты = ?(ЗначениеЗаполнено(Запись_мВалюта.Кратность), Запись_мВалюта.Кратность, 1);
			
		Иначе 
			
			Курс_мВалюты 	  = 1;
			Кратность_мВалюты = 1;
			
		КонецЕсли;	
		
		КоэффициентПересчета = (Курс_мВалюты) / (Кратность_мВалюты);
		
		НаборЗаписей = РегистрыБухгалтерии.Хозрасчетный.СоздатьНаборЗаписей();
		НаборЗаписей.Отбор.Регистратор.Установить(НовыйДокумент.Ссылка);
		НаборЗаписей.Прочитать();
		
		Баланс = Число(КоллекцияДанных.ДополнительныеСвойства.Баланс);
		
		Если Баланс <> 0 Тогда
			
			ВыполнитьДвиженияЦессий(ВалютаДт, ВалютаКт, ВалютныйСчет, КоэффициентПересчета, НаборЗаписей, НовыйДокумент, Баланс);
						
		КонецЕсли;
		
		Переоценка = Число(КоллекцияДанных.ДополнительныеСвойства.Переоценка);
		
		Если Переоценка <> 0 Тогда
			
			ВыполнитьДвиженияЦессий(ВалютаДт, ВалютаКт, ВалютныйСчет, КоэффициентПересчета, НаборЗаписей, НовыйДокумент, Переоценка);
						
		КонецЕсли;
		
		Если НаборЗаписей.Количество() > 1 Тогда 
			
			НаборЗаписей.ОбменДанными.Загрузка = Истина;
			
			НаборЗаписей.Записать();
			
			НовыйДокумент.ОбменДанными.Загрузка = Истина;
			НовыйДокумент.РучнаяКорректировка   = Истина;
			НовыйДокумент.Записать();
			
		КонецЕсли;	
		
		ЗарегистрироватьОбъектИмпорта(КоллекцияДанных.ДополнительныеСвойства.ИдентификаторЗаключения, КоллекцияДанных.ДополнительныеСвойства.ИдентификаторДопСобытия, НовыйДокумент.Ссылка);
		
		ЗафиксироватьТранзакцию();
		
		НомерИтерации = НомерИтерации + 1;
		
	КонецЦикла;
	
КонецПроцедуры
