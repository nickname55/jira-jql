https://developer.atlassian.com/server/jira/platform/adding-a-jql-function-to-jira/

Добавление JQL функции в Jira.

Применимо для Джира 7 или новее.

В этом руководстве показано,
как добавить функцию JQL в Jira.
После этого вы сможете использовать
функцию в форме расширенного поиска (advanced search),
чтобы находить задачи в проектах
к которым вы недавно обращались.
В реальном мире пользователь,
скорее всего,
использовал бы эту функциюв соченании с другими операторами поиска.
Это было бы полезно для систем,
где есть множество проектов,
и где пользователи
обычно пользуются только несколькими из них.

Запрост JQL состоит из одного или нескольких условий.
Каждое условие состоит из поля, оператора и операнда.
Например, assignee = fred.

где field (поле) это assignee
оператор это = 
операнд это fred

В данном случае операнд - это непосредственно строка "fred"

Но на месте операнда также может находиться и другая функция.

Jira имеет множество встроенных функций.

И вы можете добавлять собственный, как мы это и сделаем сейчас здесь.

В этом руководстве вы создадите функцию JQL,
которая будет состоять из следующих компонентов:

Классов Java, которые будут содержать логику.
Дескриптор плагина, для включения этой логики в состав Jira.
Когда вы закончите, то эти компоненты будут упакованы в один файл JAR.

================

Вы можете использовать любую поддерживаемую компбинацию
операционной системы и среды разработки (IDE)
для того чтобы создать этот плагин с JQL функцией.

Это руководство последний раз тестировалось
для Jira 7.7.1
AMPS 6.3.15 и Atlassian SDK 6.3.10

Прежде чем вы начнете
Для выполнения всех шагов руководства вам необходимо
знать следующее:

основы разработки с использованием Java: классы, интерфейсы, методы,
использование компилятора и так далее.
Как создать проект плагина для Atlassian Jira
при помощи Atlassian Plugin SDK.
Основы использования и администрирования Jira,
а также способы использования расширенного поиска JQL :
https://confluence.atlassian.com/display/JIRACORESERVER/Advanced+searching

## Исходные коды плагина

Мы рекомендуем вам проработать это руководство,
если вы хотите пропустить какие-то шаги
или проверить правильно ли вы их сделали,
то можете посмотреть исходный код расположенный 
в репозитории. 
Просто клонируйте репу:
git clone https://bitbucket.org/atlassian_tutorial/jira-simple-jql-function.git

Кроме того вы можете скачать исходники в виде zip архива
https://bitbucket.org/atlassian_tutorial/jira-simple-jql-function/get/master.zip

## Шаг 1 - создание проекта

В этом шаге вы будете использовать команды Atlassian SDK,
поэтому предварительно установить его если он еще не установлен.

Эти команды начинаются с префикса atlas-
и доступны в командной строке.

Они автоматизируют для вас большую часть работы 
по разработке плагинов для Jira и других продуктов Atlassian.

Выполните следующую команду
чтобы сгенерировать шаблон проекта

atlas-create-jira-plugin

у вас запросят следующую информацию

group-id,
artifact-id,
version
package

введите соответственно
com.example.plugins.tutorial.jira	
jira-simple-jql-function
1.0-SNAPSHOT
com.example.plugins.tutorial.jira

затем вас попросят подтвердить что все введенные данные правильны.

SDK создаст директорию с исходным кодом плагина 
Имя директории 
jira-simple-jql-function

## Немного настроим POM

Рекомендуем ознакомиться с файлом конфигурации проекта,
pom.xml (файлом определения Project Object Model).

На этом этапе мы просмотрим и настроим файл pom.xml
В pom.xml объявляются зависимости проекта и другая информация.

Перейдите в директорию jira-simple-jql-function и откройте файл pom.xml.

Добавьте название своей компании или организации и URL-адрес вашего веб-сайта
в элемент organization (следующий блок кода показывает, как это выглядит):

``` xml
<organization>
    <name>Example Company</name>
    <url>http://www.example.com/</url>
</organization>
```

Обновите description элемент

```
<description>Adds a custom JQL function named recentProjects to JIRA.</description>
```

Сохраните и закройте файл.

## Добавим в плагин модули

Теперь мы воспользуемся генератором plugin module generator
(для этого используется другая команда atlas- )

Нам нужен специальный модуль для функции JQL

Перейдите в корень проекта и 
введи в терминале команду

atlas-create-jira-plugin-module

Выберите опцию JQL Function.

Затем в ответ на запрос, введите следующую информацию:

Имя класса (class name)
RecentProjectFunction

Имя пакета (package name)
com.example.plugins.tutorial.jira.jql

Выберите N чтобы откзааться от показа расширенных настроеки (Show Advanced Setup)
Выберите N чтобы отказаться от добавление еще одного модуля (Add Another Plugin Module)

Подтвердите свой выбор.

## Шаг 4 - настроим дестриптор плагина

SDK добавил в дескриптор нашего приложения
модуль JQL Function.
Дескриптор приложения описывает ваш плагин для успешного его импорта в Jira.

Перейдите в src/main/resources/
и откройте файл atlassian-plugin.xml 

Найдите элемент jql-function,
а затем добавьте элементы fname и list 
за элементом description

```
<jql-function name="Recent Project Function"
              i18n-name-key="recent-project-function.name"
              key="recent-project-function"
              class="com.example.plugins.tutorial.jira.jql.RecentProjectFunction">
  <description key="recent-project-function.description">The Recent Project Function Plugin</description>
  <fname>recentProjects</fname>
  <list>true</list>
</jql-function>
```

Имя fname представляет имя нашей функции, поскольку оно будет использоваться в операторах JQL. Список указывает, возвращает ли эта функция список проблем или одно значение.

Сохраните и закройте файл.

## Шаг 5 - Напишите код приложения

SDK предоставляет нам stub code (заглушку)
для нашего класса.

Давайте добавим логику для нашей функции.

Перейдите в src/main/java/com/example/plugins/tutorial/jira/jql/
и откройте файл RecentProjectFunction.java

Замените import com.opensymphony.user.User; 
на следующую строку:
import com.atlassian.jira.user.ApplicationUser;

Обновите метод validate(),
чтобы использовать ApplicationUser вместо User

```
public MessageSet validate(ApplicationUser searcher, FunctionOperand operand, TerminalClause terminalClause) {
    return validateNumberOfArgs(operand, 1);
}
```

Добавьте следующие импорты:

```
import com.atlassian.plugin.spring.scanner.annotation.component.Scanned;
import com.atlassian.plugin.spring.scanner.annotation.imports.ComponentImport;
import com.atlassian.jira.user.UserHistoryItem;
import com.atlassian.jira.user.UserProjectHistoryManager;
import java.util.LinkedList;
```

Перед определением класса добавьте аннотацию @Scanned

```
@Scanned
public class RecentProjectFunction extends AbstractJqlFunction {
   ...
```

Чтобы определить, к каким проектам текущий пользователь
недавно обращался, мы можем использовать UserProjectHistoryManager,
этот класс нам предоставляет Jira.

Вставьте этот класс в конструктор нашего класса recentProjectFunction 
как показано ниже:

``` java
@ComponentImport
private final UserProjectHistoryManager userProjectHistoryManager;

public RecentProjectFunction(UserProjectHistoryManager userProjectHistoryManager) {
   this.userProjectHistoryManager = userProjectHistoryManager;
} 
```

Наша функция пока не принимает аргументом.
В методе validate() измените количество аргументов
с одного на ноль

``` java
return validateNumberOfArgs(operand, 0); 
```

Функция getValues() - это то место, 
где происходит большая часть работы.
Замените код, которые сгенерировал SDK
на этот :

```
public List<QueryLiteral> getValues(QueryCreationContext queryCreationContext, FunctionOperand operand, TerminalClause terminalClause) {
    final List<QueryLiteral> literals = new LinkedList<>();
    final List<UserHistoryItem> projects = userProjectHistoryManager.getProjectHistoryWithoutPermissionChecks(queryCreationContext.getApplicationUser());

    for (final UserHistoryItem userHistoryItem : projects) {
        final String value = userHistoryItem.getEntityId();

        try {
            literals.add(new QueryLiteral(operand, Long.parseLong(value)));
        } catch (NumberFormatException e) {
            log.warn(String.format("User history returned a non numeric project IS '%s'.", value));
        }
    }        
    return literals;
}
```

Функция возвращает список QueryLiterals,
которые представляют
список идентификаторов (IDs)
тех проектов, которые вы недавно посетили 

Любой пользователь может использовать эту функцию, поэтому мы используем getProjectHistoryWithoutPermissionChecks() чтобы при
использовании функции не выполнялось никаких проверок.

В качестве альтернативы мы можем использовать getProjectHistoryWithPermissionChecks(), эта функция выполняет проверку прав на основании тех разрешений, которые пользователь имеет в проектах.

В In getMinimumNumberOfExpectedArguments(), изменим
возвращаемое значение на 0:

```
public int getMinimumNumberOfExpectedArguments() {
  return 0;
}
```

В getDataType() измените возвращаемый тип данных, с TEXT на PROJECT, потому что мы возвращаем список проектов.

return JiraDataTypes.PROJECT;

Весь класс таперь доложен выглядеть примерно так:

```
package com.example.plugins.tutorial.jira.jql;

import com.atlassian.jira.user.ApplicationUser;
import com.atlassian.jira.user.UserHistoryItem;
import com.atlassian.jira.user.UserProjectHistoryManager;
import com.atlassian.plugin.spring.scanner.annotation.component.Scanned;
import com.atlassian.plugin.spring.scanner.annotation.imports.ComponentImport;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import com.atlassian.jira.JiraDataType;
import com.atlassian.jira.JiraDataTypes;
import com.atlassian.jira.jql.operand.QueryLiteral;
import com.atlassian.jira.jql.query.QueryCreationContext;
import com.atlassian.jira.plugin.jql.function.AbstractJqlFunction;
import com.atlassian.jira.util.MessageSet;
import com.atlassian.query.clause.TerminalClause;
import com.atlassian.query.operand.FunctionOperand;
import java.util.LinkedList;
import java.util.List;

@Scanned
public class RecentProjectFunction extends AbstractJqlFunction {
    private static final Logger log = LoggerFactory.getLogger(RecentProjectFunction.class);

    @ComponentImport
    private final UserProjectHistoryManager userProjectHistoryManager;

    public RecentProjectFunction(UserProjectHistoryManager userProjectHistoryManager) {
        this.userProjectHistoryManager = userProjectHistoryManager;
    }

    public MessageSet validate(ApplicationUser searcher, FunctionOperand operand, TerminalClause terminalClause) {
        return validateNumberOfArgs(operand, 0);
    }

    public List<QueryLiteral> getValues(QueryCreationContext queryCreationContext, FunctionOperand operand, TerminalClause terminalClause) {
        final List<QueryLiteral> literals = new LinkedList<>();
        final List<UserHistoryItem> projects = userProjectHistoryManager.getProjectHistoryWithoutPermissionChecks(queryCreationContext.getApplicationUser());

        for (final UserHistoryItem userHistoryItem : projects) {
            final String value = userHistoryItem.getEntityId();

            try {
                literals.add(new QueryLiteral(operand, Long.parseLong(value)));
            } catch (NumberFormatException e) {
                log.warn(String.format("User history returned a non numeric project IS '%s'.", value));
            }
        }

        return literals;
    }

    public int getMinimumNumberOfExpectedArguments() {
        return 0;
    }

    public JiraDataType getDataType() {
        return JiraDataTypes.PROJECT;
    }
}
```

## Шаг 6 - удалите тестовые файлы

SDK пытается быть полезным,

поэтому при генерации проекта

предоставляет нам файлы-заглушки для модульных и интеграционных тестовые

для кода нашего приложения.

Тем не менее,

они нам сейчас не нужны,

и давайте просто удалим им чтобы просто проверить как работае наш плагин.

Для этого в терминале введите команду:

```
rm -rf src/main/test
```

## Запустим Jira и протестируем работу плагина

Откройте окно терминала,

перейдите в корневую директорию плагина

и запустите команду atlas-run

(или atlas-debug, если вы хотите запустить отладчик

в своей среде разработки )

Откройте Jira в браузере

(URL адрес указан вы выводе терминала)

Войдите в систему, используя логин admin
и пароль admin

Создайте в Jira 2 или 3 проекта

и в этих проектах создайте несколько задач.

Это будут данные при помощи которых

мы сможем протестировать работу нашей JQL-функции

В шапке Jira-страницы нажмите Issues -> Search for issues

Чтобы переключить форму в расширенный режим (advanced mode)

Нажмите Advanced 

В поле поиска введите следующее

```
project in recentProjects() 
```

Обратите внимание,

что autocomplete 

предлагает новую функцию в строке автозавершения вашего JQL-запроса.

Щелкните по значку поиска, чтобы выполнить запрос.

Появится список задач, которые находятся в недавно посещенных вами проектах.
