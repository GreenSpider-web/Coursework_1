# College Database System (Курсова робота)

## 🛠 Технологічний стек
* **Java 25 / Maven**
* **ORM:** Hibernate Core
* **БД:** PostgreSQL
* **HTTP Server:** Власний кастомний "фреймворк" на базі `com.sun.net.httpserver`
* **Автентифікація & Безпека:** Session-based (Cookies + ConcurrentHashMap), BCrypt
* **Фронтенд:** Vanilla JS / HTML5 / CSS3

---

## 🏗 Архітектура та структура проєкту
```
university-db/
├── pom.xml                       # hibernate-core, mysql-connector-j, gson, jbcrypt
│                                 # + maven-shade-plugin (fat-jar для java -jar)
└── src/main/
    ├── java/ua/edu/university/
    │   ├── Main.java                         # створює HttpServer, реєструє маршрути, start()
    │   │
    │   ├── config/
    │   │   ├── AppConfig.java                # порт, налаштування з application.properties
    │   │   └── HibernateUtil.java            # один SessionFactory
    │   │
    │   ├── entity/                           # без змін
    │   │   ├── enums/   Role, StudyForm, ClassroomType, LessonType, WeekType,
    │   │   │            AttendanceStatus, FinalStatus
    │   │   ├── User  Faculty  Department  Specialty  StudyGroup
    │   │   ├── Teacher  Student  Classroom
    │   │   ├── Subject  Course  Lesson
    │   │   └── Assignment  Grade  Attendance  FinalGrade
    │   │
    │   ├── dao/                              # без змін
    │   │   ├── GenericDao.java
    │   │   ├── AbstractHibernateDao.java
    │   │   └── impl/   UserDao, StudentDao, TeacherDao, GroupDao, ClassroomDao,
    │   │               CourseDao, LessonDao, GradeDao, AttendanceDao, FinalGradeDao ...
    │   │
    │   ├── service/                          # без змін: логіка + транзакції
    │   │   ├── AuthService  StudentService  TeacherService  ClassroomService
    │   │   ├── CourseService  ScheduleService  GradeService
    │   │   └── AttendanceService  FinalGradeService
    │   │
    │   ├── dto/
    │   │   ├── request/    LoginRequest, StudentForm, TeacherForm, GradeUpdateRequest,
    │   │   │               AttendanceMarkRequest ...
    │   │   └── response/   StudentDto, TeacherDto, JournalDto, ScheduleCellDto, ApiError
    │   │
    │   ├── http/                             # власний мінімальний "фреймворк"
    │   │   ├── Router.java                   # (метод + шаблон шляху) → обробник, витягує {id}
    │   │   ├── Route.java
    │   │   ├── HttpMethod.java
    │   │   ├── RequestContext.java           # обгортка HttpExchange: path/query-параметри,
    │   │   │                                 # тіло → об'єкт, поточний користувач
    │   │   ├── Response.java                 # json(), html(), redirect(), status()
    │   │   ├── filter/                       # com.sun.net.httpserver.Filter
    │   │   │   ├── LoggingFilter.java
    │   │   │   └── AuthFilter.java           # сесія з cookie + перевірка ролі за префіксом URL
    │   │   ├── session/
    │   │   │   ├── SessionManager.java       # ConcurrentHashMap<sid, UserSession>, cookie SID
    │   │   │   └── UserSession.java
    │   │   ├── json/
    │   │   │   └── JsonMapper.java           # Gson + адаптери LocalDate / LocalDateTime
    │   │   └── exception/
    │   │       ├── HttpException.java        # базовий, зі статус-кодом
    │   │       ├── BadRequestException  NotFoundException  ForbiddenException
    │   │       └── ExceptionMapper.java      # виняток → JSON {error} з потрібним кодом
    │   │
    │   ├── handler/                          # замість сервлетів
    │   │   ├── StaticFileHandler.java        # /static/** з resources, MIME-типи
    │   │   ├── PageHandler.java              # /admin/students → pages/admin/students.html
    │   │   └── api/
    │   │       ├── AuthApiHandler            ├── CourseApiHandler
    │   │       ├── StructureApiHandler       ├── ScheduleApiHandler
    │   │       ├── GroupApiHandler           ├── AssignmentApiHandler
    │   │       ├── StudentApiHandler         ├── GradeApiHandler
    │   │       ├── TeacherApiHandler         ├── AttendanceApiHandler
    │   │       ├── ClassroomApiHandler       └── FinalGradeApiHandler
    │   │       └── SubjectApiHandler
    │   │
    │   └── util/   PasswordUtil (BCrypt), EctsCalculator, Validator
    │
    └── resources/
        ├── application.properties            # server.port, db url/user/password
        ├── hibernate.cfg.xml                 # hbm2ddl.auto=validate
        ├── db/   schema.sql   seed.sql
        └── web/                              # фронтенд, вшитий у jar
            ├── pages/
            │   ├── login.html  home.html  profile.html  schedule.html
            │   ├── admin/     dashboard  structure  groups  students  teachers
            │   │              classrooms  subjects  courses  schedule-editor  (.html)
            │   ├── teacher/   courses  course  journal  attendance  final-grades  (.html)
            │   ├── student/   schedule  grades  attendance  transcript  (.html)
            │   └── error/     403.html  404.html
            └── static/
                ├── css/style.css
                └── js/
                    ├── api.js                # обгортка fetch; при 401 → /login
                    ├── layout.js             # шапка й меню під роль з /api/auth/me
                    └── pages/                # по скрипту на сторінку: students.js, journal.js ...
```
