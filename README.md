# Лабораторная работа № 2 Разработка безопасных веб-приложений
> Выполнили студенты группы РИ-571227 Мартынов Артём, Бушуев Антон

В изначальном уязвимом приложении SQL инъекцию можно было осуществить при сортировке названий книг на сайте путём подстановки кавычки:

![Image alt](https://github.com/ArtemMartynov/screens/raw/main/LR_2_Web_screen_1.png)

Для обхода фильтра использовался запрос на получение названий таблиц с помощью UNION, что представлено ниже:

Запрос: asdfgdh%' union all SELECT NULL, table_name, table_catalog FROM information_schema.tables; --

Результат запроса:

![Image alt](https://github.com/ArtemMartynov/screens/raw/main/LR_2_Web_screen_2.png)

Так как теперь получены имена таблиц, то можно запросить значения, например, из таблицы users:

Запрос: asdfgdh%' union all SELECT NULL, * FROM users; --

Результат:

![Image alt](https://github.com/ArtemMartynov/screens/raw/main/LR_2_Web_screen_4.png)

Т.е. теперь можно спокойно похитить пароли пользователей.

Для Исправления уязвимости сделано следующее:

Часть кода


``app.get('/books', async (req, res) => {
    let bookname = req.query.name;    
    let sql = select ba.id, a.name as author, b.name as book from books_by_authors ba    
                  left join author a on a.id = ba.aid                  
                  left join book b on b.id = ba.bid;                  
    if(bookname){    
        sql+=\rwhere b.name like '%${bookname}%'        
    };``

Заменена на

``app.get('/books', async (req, res) => {
    let bookname = '';	  
    if (req.query.name){    
        	bookname+= req.query.name};  
    let sql = {    
        		text:select ba.id, a.name as author, b.name as book from books_by_authors ba  
                  left join author a on a.id = ba.aid                  
                  left join book b on b.id = ba.bid where b.name like $1,                  
            values: ['%'+ bookname + '%']            
                  };``
                  
В результате инъекция с использованием кавычек больше не работает:

![Image alt](https://github.com/ArtemMartynov/screens/raw/main/LR_2_Web_screen_6.png)
