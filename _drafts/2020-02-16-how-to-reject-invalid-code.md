---
layout: post
title: Как не пропустить невалидный код в репозиторий
category: development
tags: [.net, processes, cicd, pipeline]
---

Когда в твоей команде работают больше одного человека, так или иначе все сталкиваются с проблемой разных стилей кодирования каждого члена команды. Кто-то пишет скобки для блоков `if-else`, кто-то нет. Когда проект становится больше, то такой код труднее читать и еще сложнее проводить код-ревью. Чтобы код-ревью и прочие командные митинги не превратились в обсуждение _tab vs spaces_ на повышенных тонах, лучше настроить репозиторий таким образом, чтобы саам проект не допускал написание невалидного и нестандартного для команды кода.

С одной стороны, использование разных стилей кодирования может показаться вкусовщиной, недостойной внимани. Ну не оборачивает джун единственную строку кода после условия `if`, а кто-то пишет, что с того? Если оставить код из под пера джуна "как есть", то это может стать "бомбой замедленного действия": ту строку кода после `if` могут удалить, и тогда под условие попадет следующая строка. Конечно, эта ситуация обычно отлавливается на код-ревью, однако бывает так, что этот потенциальный баг проходит проверку, и вот две основных причины:

1. Мы все люди, а люди ошибаются.
2. Люди социальны, а значит вступать "в очередной раз" в конфликт, связанный со стилями, не захочется. И тут возможны два варианта:
    - "Лучше поправлю сам", думает проверяющий, и правит код.
    - Проверяющий срывается на джуна и высказывает свои сомнения в его адекватности и необходимости существования.

Как можно добиться того, чтобы каждый писал в соответствии с командным стилем? Бить по рукам на код-ревью каждый раз демотивирует и автора кода, и самого проверяющего. К счастью, эта проблема будоражит умы не одного программиста не первый год, и в нашем распоряжении сейчас есть множество инструментов.

Цель этой статьи - рассказать другим и себе будущему, как я настраиваю репозиторий проекта таким образом, чтобы он сам обезопасил себя от невалидного кода с точки зрения стандартов команды.

# Что мы имеем

В качестве примера возьмем демонстрационный проект, код которого будет выложен на GitHub. Так как я занимаюсь разработкой на .NET Core, то и проект будет написан на нем. Что я буду использовать:

- .NET Core 3.1
- Angular 8+
- Github аккаунт
- Travis CI

# Пайплайн репозитория

Пайплайн репозитория - механизм, предотвращающий попадаение невалидного кода с второстепенных веток в основную `master branch`. Сейчас такую возможность дают системы [Gitlab](https://gitlab.com), [GitHub](https://github.com) и [Azure DevOps](https://dev.azure.com). О других системах репозиториев я не знаю, но для работы мне пока и не нужно было. Я сам пользуюсь GitLab, потому что помимо пайплайнов в гитлабе удобно проводить код-ревью, а также настраивать правила и ограничения пушей в ветки.

## Настраиваем репозиторий

Мне нравится подход к разработке софта [Егора Бугаенко](https://yegor256.com). Я законспектировал [несколько его докладов](/menu/tags.html#yegor256) на этом блоге. Если кратко, то вот основные принципы, которым я буду следовать при настройке репозитория:

- **Ограничение прав на пуш**. Я ограничу права на пуш в develop и master всем, кроме мейнтейнеров (maintainer).
- **Пайплайн сборки**. Я пропишу пайплайн для сборки проекта в гитлаб и прогона юниттестов как для бэкенда, так и фронта.
- **Repository is a king**. В репозитории будут прописаны правила работы с кодом и gitflow, а также другие связанные с подходами в разработке документы.
- **Fail fast**. Если код написан невалидно с точки зрения стандартного стиля, то разработчик получит ошибку компиляции.
- **Git pre-commits hoocks**. Чтобы не занимать раннеры гитлаба лишком часто, я добавлю прогон тестов и иные полезные операции на пре-коммит хуки гита.

Что мы получим в итоге? Во-первых, в master и develop смогут залить код только мейнтейнеры проекта. В идеале, конечно, и им нужно ограничить доступ, чтобы только "автомат" мог сливать ветки. Я оставил реализацию этого принципа "на потом". Ограничение прав настраивается через интерфейс гитлаба, поэтому я не буду описывать этот этап здесь.

## Валидация бэкенда

Я настраиваю solution-файл (*.sln) проекта так, чтобы он выдавал несоответствия написанного кода стандартам стайл-гайда .NET как ошибки компиляции. Чтобы сделать это, мне понадобится файл с перечислением кодов ошибок, пара nuget-пакетов и немного терпения.

Я использую stylecop в проектах для .NET Core. Чтобы его верно настроить, прежде всего мы создаем несколько файлов в корне проекта рядом с solution-файлом:

1. `Directory.build.props`:

<details>
<summary>Много текста</summary>

  ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <Project>

      <PropertyGroup>
        <Description>An implementation of StyleCop rules using the .NET Compiler Platform</Description>
        <Product>StyleCop.Analyzers</Product>
        <Company>Tunnel Vision Laboratories, LLC</Company>
        <Copyright>Copyright © Tunnel Vision Laboratories, LLC 2015</Copyright>
        <NeutralLanguage>en-US</NeutralLanguage>

        <Version>1.1.0.39</Version>
        <FileVersion>1.1.0.39</FileVersion>
        <InformationalVersion>1.1.0-dev</InformationalVersion>
      </PropertyGroup>

      <PropertyGroup>
        <LangVersion>7</LangVersion>
        <Features>strict</Features>
      </PropertyGroup>

      <PropertyGroup Condition="'$(BuildingInsideVisualStudio)' != 'true'">
        <TreatWarningsAsErrors>true</TreatWarningsAsErrors>
      </PropertyGroup>

      <PropertyGroup>
        <DebugType>portable</DebugType>
        <DebugSymbols>true</DebugSymbols>
      </PropertyGroup>

      <PropertyGroup>
        <GenerateDocumentationFile>False</GenerateDocumentationFile>
        <NoWarn>$(NoWarn),1573,1591,1712</NoWarn>
      </PropertyGroup>

      <ItemGroup Condition="'$(DisableStylecop)' != true">
        <PackageReference Include="AsyncUsageAnalyzers" Version="1.0.0-alpha003" PrivateAssets="all" />
        <PackageReference Include="StyleCop.Analyzers" Version="1.1.118" PrivateAssets="all" IncludeAssets="runtime; build; native; contentfiles; analyzers"/>
      <AdditionalFiles Include="$(MSBuildThisFileDirectory)stylecop.json">
          <Link>stylecop.json</Link>
        </AdditionalFiles>
      </ItemGroup>

    <ItemGroup>
        <None Include="$(MSBuildThisFileDirectory)standard.ruleset" Link="standard.ruleset" />
        <None Include="$(AppDesignerFolder)\launchSettings.json" Condition="Exists('$(AppDesignerFolder)\launchSettings.json')" />
      </ItemGroup>

      <PropertyGroup>
        <CodeAnalysisRuleSet>$(MSBuildThisFileDirectory)standard.ruleset</CodeAnalysisRuleSet>
      </PropertyGroup>
      
  </Project>
  ```
</details>



2. `standard.ruleset`:

<details>
  <summary>Слишком много строк</summary>
  ```xml
<?xml version="1.0" encoding="utf-8"?>
<RuleSet Name="Standard Rule Set" Description=" " ToolsVersion="16.0">
    <Rules AnalyzerId="StyleCop.Analyzers" RuleNamespace="StyleCop.Analyzers">
        <Rule Id="SA1001" Action="Error" />
        <Rule Id="SA1002" Action="Error" />
        <Rule Id="SA1003" Action="Error" />
        <Rule Id="SA1122" Action="Error" />
        <Rule Id="SA1000" Action="Error" />
        <Rule Id="SA0000" Action="Hidden" />
        <Rule Id="CA1001" Action="Error" />
        <Rule Id="CA1009" Action="Error" />
        <Rule Id="CA1016" Action="Error" />
        <Rule Id="CA1033" Action="Error" />
        <Rule Id="CA1049" Action="Error" />
        <Rule Id="CA1060" Action="Error" />
        <Rule Id="CA1061" Action="Error" />
        <Rule Id="CA1063" Action="Error" />
        <Rule Id="CA1065" Action="Error" />
        <Rule Id="CA1301" Action="Error" />
        <Rule Id="CA1400" Action="Error" />
        <Rule Id="CA1401" Action="Error" />
        <Rule Id="CA1403" Action="Error" />
        <Rule Id="CA1404" Action="Error" />
        <Rule Id="CA1405" Action="Error" />
        <Rule Id="CA1410" Action="Error" />
        <Rule Id="CA1415" Action="Error" />
        <Rule Id="CA1821" Action="Error" />
        <Rule Id="CA1900" Action="Error" />
        <Rule Id="CA1901" Action="Error" />
        <Rule Id="CA2002" Action="Error" />
        <Rule Id="CA2100" Action="Error" />
        <Rule Id="CA2101" Action="Error" />
        <Rule Id="CA2108" Action="Error" />
        <Rule Id="CA2111" Action="Error" />
        <Rule Id="CA2112" Action="Error" />
        <Rule Id="CA2114" Action="Error" />
        <Rule Id="CA2116" Action="Error" />
        <Rule Id="CA2117" Action="Error" />
        <Rule Id="CA2122" Action="Error" />
        <Rule Id="CA2123" Action="Error" />
        <Rule Id="CA2124" Action="Error" />
        <Rule Id="CA2126" Action="Error" />
        <Rule Id="CA2131" Action="Error" />
        <Rule Id="CA2132" Action="Error" />
        <Rule Id="CA2133" Action="Error" />
        <Rule Id="CA2134" Action="Error" />
        <Rule Id="CA2137" Action="Error" />
        <Rule Id="CA2138" Action="Error" />
        <Rule Id="CA2140" Action="Error" />
        <Rule Id="CA2141" Action="Error" />
        <Rule Id="CA2146" Action="Error" />
        <Rule Id="CA2147" Action="Error" />
        <Rule Id="CA2149" Action="Error" />
        <Rule Id="CA2200" Action="Error" />
        <Rule Id="CA2202" Action="Error" />
        <Rule Id="CA2207" Action="Error" />
        <Rule Id="CA2212" Action="Error" />
        <Rule Id="CA2213" Action="Error" />
        <Rule Id="CA2214" Action="Error" />
        <Rule Id="CA2216" Action="Error" />
        <Rule Id="CA2220" Action="Error" />
        <Rule Id="CA2229" Action="Error" />
        <Rule Id="CA2231" Action="Error" />
        <Rule Id="CA2232" Action="Error" />
        <Rule Id="CA2235" Action="Error" />
        <Rule Id="CA2236" Action="Error" />
        <Rule Id="CA2237" Action="Error" />
        <Rule Id="CA2238" Action="Error" />
        <Rule Id="CA2240" Action="Error" />
        <Rule Id="CA2241" Action="Error" />
        <Rule Id="CA2242" Action="Error" />
        <Rule Id="SA1004" Action="Error" />
        <Rule Id="SA1005" Action="Error" />
        <Rule Id="SA1006" Action="Error" />
        <Rule Id="SA1007" Action="Error" />
        <Rule Id="SA1008" Action="Error" />
        <Rule Id="SA1009" Action="Error" />
        <Rule Id="SA1010" Action="Error" />
        <Rule Id="SA1011" Action="Error" />
        <Rule Id="SA1012" Action="Error" />
        <Rule Id="SA1013" Action="Error" />
        <Rule Id="SA1014" Action="Error" />
        <Rule Id="SA1015" Action="Error" />
        <Rule Id="SA1016" Action="Error" />
        <Rule Id="SA1017" Action="Error" />
        <Rule Id="SA1018" Action="Error" />
        <Rule Id="SA1019" Action="Error" />
        <Rule Id="SA1020" Action="Error" />
        <Rule Id="SA1021" Action="Error" />
        <Rule Id="SA1022" Action="Error" />
        <Rule Id="SA1023" Action="Error" />
        <Rule Id="SA1300" Action="Error" />
        <Rule Id="SA1127" Action="Error" />
        <Rule Id="SA1128" Action="Error" />
        <Rule Id="SA1129" Action="Error" />
        <Rule Id="SA1130" Action="Error" />
        <Rule Id="SA1131" Action="Error" />
        <Rule Id="SA1132" Action="Error" />
        <Rule Id="SA1133" Action="Error" />
        <Rule Id="SA1134" Action="Error" />
        <Rule Id="SA1200" Action="Error" />
        <Rule Id="SA1201" Action="Error" />
        <Rule Id="SA1202" Action="Error" />
        <Rule Id="SA1203" Action="Error" />
        <Rule Id="SA1204" Action="Error" />
        <Rule Id="SA1205" Action="Error" />
        <Rule Id="SA1206" Action="Error" />
        <Rule Id="SA1207" Action="Error" />
        <Rule Id="SA1208" Action="Error" />
        <Rule Id="SA1209" Action="Error" />
        <Rule Id="SA1210" Action="Error" />
        <Rule Id="SA1211" Action="Error" />
        <Rule Id="SA1212" Action="Error" />
        <Rule Id="SA1213" Action="Error" />
        <Rule Id="SA1214" Action="Error" />
        <Rule Id="SA1216" Action="Error" />
        <Rule Id="SA1217" Action="Error" />
        <Rule Id="SA1024" Action="Error" />
        <Rule Id="SA1025" Action="Error" />
        <Rule Id="SA1026" Action="Error" />
        <Rule Id="SA1027" Action="Error" />
        <Rule Id="SA1028" Action="Error" />
        <Rule Id="SA1100" Action="Error" />
        <Rule Id="SA1101" Action="None" />
        <Rule Id="SA1102" Action="Error" />
        <Rule Id="SA1103" Action="Error" />
        <Rule Id="SA1104" Action="Error" />
        <Rule Id="SA1105" Action="Error" />
        <Rule Id="SA1106" Action="Error" />
        <Rule Id="SA1107" Action="Error" />
        <Rule Id="SA1108" Action="Error" />
        <Rule Id="SA1110" Action="Error" />
        <Rule Id="SA1111" Action="Error" />
        <Rule Id="SA1112" Action="Error" />
        <Rule Id="SA1113" Action="Error" />
        <Rule Id="SA1114" Action="Error" />
        <Rule Id="SA1115" Action="Error" />
        <Rule Id="SA1116" Action="Error" />
        <Rule Id="SA1117" Action="Error" />
        <Rule Id="SA1118" Action="None" />
        <Rule Id="SA1119" Action="Error" />
        <Rule Id="SA1120" Action="Error" />
        <Rule Id="SA1121" Action="Error" />
        <Rule Id="SA1123" Action="Error" />
        <Rule Id="SA1124" Action="Error" />
        <Rule Id="SA1125" Action="Error" />
        <Rule Id="SA1302" Action="Error" />
        <Rule Id="SA1303" Action="Error" />
        <Rule Id="SA1304" Action="Error" />
        <Rule Id="SA1305" Action="None" />
        <Rule Id="SA1306" Action="Error" />
        <Rule Id="SA1307" Action="Error" />
        <Rule Id="SA1308" Action="Error" />
        <Rule Id="SA1309" Action="None" />
        <Rule Id="SA1310" Action="Error" />
        <Rule Id="SA1311" Action="Error" />
        <Rule Id="SA1312" Action="Error" />
        <Rule Id="SA1313" Action="Error" />
        <Rule Id="SA1400" Action="Error" />
        <Rule Id="SA1401" Action="Error" />
        <Rule Id="SA1402" Action="Error" />
        <Rule Id="SA1403" Action="Error" />
        <Rule Id="SA1404" Action="Error" />
        <Rule Id="SA1405" Action="Error" />
        <Rule Id="SA1406" Action="Error" />
        <Rule Id="SA1407" Action="Error" />
        <Rule Id="SA1408" Action="Error" />
        <Rule Id="SA1410" Action="Error" />
        <Rule Id="SA1411" Action="Error" />
        <Rule Id="SA1412" Action="Error" />
        <Rule Id="SA1413" Action="None" />
        <Rule Id="SA1500" Action="Error" />
        <Rule Id="SA1501" Action="Error" />
        <Rule Id="SA1502" Action="Error" />
        <Rule Id="SA1503" Action="Error" />
        <Rule Id="SA1504" Action="Error" />
        <Rule Id="SA1505" Action="Error" />
        <Rule Id="SA1506" Action="Error" />
        <Rule Id="SA1507" Action="Error" />
        <Rule Id="SA1508" Action="Error" />
        <Rule Id="SA1509" Action="Error" />
        <Rule Id="SA1510" Action="Error" />
        <Rule Id="SA1511" Action="Error" />
        <Rule Id="SA1512" Action="Error" />
        <Rule Id="SA1513" Action="Error" />
        <Rule Id="SA1514" Action="Error" />
        <Rule Id="SA1515" Action="Error" />
        <Rule Id="SA1516" Action="Error" />
        <Rule Id="SA1517" Action="Error" />
        <Rule Id="SA1518" Action="None" />
        <Rule Id="SA1519" Action="Error" />
        <Rule Id="SA1520" Action="Error" />
        <Rule Id="SA1600" Action="None" />
        <Rule Id="SA1601" Action="None" />
        <Rule Id="SA1602" Action="None" />
        <Rule Id="SA1604" Action="Error" />
        <Rule Id="SA1605" Action="Error" />
        <Rule Id="SA1606" Action="Error" />
        <Rule Id="SA1607" Action="Error" />
        <Rule Id="SA1608" Action="Error" />
        <Rule Id="SA1609" Action="None" />
        <Rule Id="SA1610" Action="Error" />
        <Rule Id="SA1611" Action="Error" />
        <Rule Id="SA1612" Action="Error" />
        <Rule Id="SA1613" Action="Error" />
        <Rule Id="SA1614" Action="Error" />
        <Rule Id="SA1615" Action="Error" />
        <Rule Id="SA1616" Action="Error" />
        <Rule Id="SA1617" Action="Error" />
        <Rule Id="SA1618" Action="Error" />
        <Rule Id="SA1619" Action="Error" />
        <Rule Id="SA1620" Action="Error" />
        <Rule Id="SA1621" Action="Error" />
        <Rule Id="SA1622" Action="Error" />
        <Rule Id="SA1623" Action="Error" />
        <Rule Id="SA1624" Action="Error" />
        <Rule Id="SA1625" Action="Error" />
        <Rule Id="SA1626" Action="Error" />
        <Rule Id="SA1627" Action="Error" />
        <Rule Id="SA1633" Action="None" />
        <Rule Id="SA1634" Action="None" />
        <Rule Id="SA1635" Action="None" />
        <Rule Id="SA1636" Action="None" />
        <Rule Id="SA1637" Action="None" />
        <Rule Id="SA1638" Action="None" />
        <Rule Id="SA1639" Action="None" />
        <Rule Id="SA1640" Action="None" />
        <Rule Id="SA1641" Action="None" />
        <Rule Id="SA1642" Action="None" />
        <Rule Id="SA1643" Action="None" />
        <Rule Id="SA1652" Action="None" />
        <Rule Id="SA1649" Action="None" />
        <Rule Id="SA1137" Action="Error" />
        <Rule Id="SA1629" Action="Error" />
        <Rule Id="SA0001" Action="None" />
        <Rule Id="CS1574" Action="None" />
    </Rules>
</RuleSet>
```
</details>


3. `stylecop.json`:

```json
{
    "$schema": "https://raw.githubusercontent.com/DotNetAnalyzers/StyleCopAnalyzers/master/StyleCop.Analyzers/StyleCop.Analyzers/Settings/stylecop.schema.json",
    "settings": {
      "documentationRules": {
        "companyName": "Petrel AI",
        "xmlHeader": false,
        "fileNamingConvention": "metadata"
      },
      "orderingRules": {
        "usingDirectivesPlacement": "outsideNamespace",
        "elementOrder": [
          "constant",
          "readonly"
        ]
      },
      "layoutRules": {
        "newlineAtEndOfFile": "require"
      }
    }
  }
```

После этих действий наш проект не будет собираться, пока в нем будут ошибки стиля кодирования.

## Валидация фронтенда

Фронтенд-приложение тоже необходимо валидировать. Здесь настройки пайплайна менее критичны к нарущениям стайл-гайда: если мы пропустим где-то точку с запятой, то проект все равно будет работать. На страже репозитория здесь будет стоять раннер (runner) пайплайна в удаленном репозитории. Я автоматизирую следующие команды:

```bash

# Проверка линта
ng lint

# Сборка в режиме продакшна, чтобы провалидировать и html-файлы
ng build --prod

# Прогон тестов
ng test

```

Есть небольшой нюанс работы раннеров репозитория с тестами. Дело в том, что для прогона тестов необходим движок Хрома (Chrome / Chromium), а он чаще всего отсутствует в раннерах. Чтобы раннер мог запускать тесты фронта, я добавляю npm-пакет `puppeteer` в проект, который подтянет с собой и хромиум.

Таким образом, чтобы и корректность фронтенда валидировалась пайплайном, нам необходимо проделать следуюзие шаги:

1. Добавить новую команду `"test-headless-ci-only": "ng test --browsers ChromiumNoSandbox"` в блок `scripts` файла `packages.json`:

```json
"scripts": {
    "ng": "ng",
    "start": "ng serve -o",
    "build": "ng build",
    "build-stage": "ng build --configuration=staging",
    "build-prod": "ng build --prod",
    "test": "ng test",
    "test-headless-ci-only": "ng test --browsers ChromiumNoSandbox",
    "lint": "ng lint",
    "e2e": "ng e2e"
  },

```

2. Установить пакет `npm install puppeteer` и прописать его в файле `karma.conf.js` в самое начало файла:

```js
const process = require("process");
process.env.CHROME_BIN = require("puppeteer").executablePath();

module.exports = function(config) {
  ...
};
```

3. Добавить кастомный лаунчер тестов в файле `karma.conf.js` в секцию `customLaunchers`:

```js
config.set({
....,
customLaunchers: {
      ChromiumNoSandbox: {
        base: "ChromeHeadless",
        flags: [
          "--no-sandbox",
          "--headless",
          "--disable-gpu",
          "--disable-translate",
          "--disable-extensions"
        ]
      }
    },
    singleRun: true
});
```

Теперь в скриптах пайплайна можно запускать тесты командой `npm test-headless-ci-only`.

## Стандартизируем код фронтенда

Чтобы код-ревью тикетов для фронтенда не превратились в обсуждение `tab vs spaces`, лучше всего стандартизировать форматирование. Лучше всего подойдет [prettierrc](https://prettier.io/docs/en/configuration.html). Чтобы добавить prettierrc в проект, необходимо:

1. Установить пакеты prettier и pretty-quick глобально:

```bash
npm install -g prettier
npm install -g pretty-quick
```

2. Добавить файл конфигурации с именем `.prettierrc` в корень фронтенд-приложения:

```json

{
    "useTabs": false,
    "printWidth": 120,
    "tabWidth": 2,
    "singleQuote": true,
    "trailingComma": "none",
    "semi": true
}

```

3. Добавить список файлов для игнорирования prettier-ом в файл с именем `.prettierignore`  в корень фронтенд-приложения:

```plaintext
package.json
package-lock.json
tslint.json
tsconfig.json
browserslist
.gitkeep
favicon.ico
tsconfig.lib.json
tsconfig.app.json
tsconfig.spec.json
karma.conf.js
protractor.conf.js
ng-package.json
*.html
```

Теперь можно "привести в порядок" код фронтенда командой `pretty-quick --staged`.

## Использование прекоммит-хуков

Запуск раннера пайплайна в CI/CD системах - это потребление ресурсов, и зачастую небесплатных. Можно и нужно запускать валидацию проекта локально, но делать это на каждый коммит надоедает. В итоге люди перестают запускать скрипты так часто. Чтобы автоматизировать этот процесс, я пользуюсь прекоммит-хуками, которые позволяют запускать полезные скрипты при коммитах и пушах. 

Для фронтенда лучше всего подойдет библиотека [husky](https://github.com/typicode/husky). Чтобы настроить хук, необходимо:

1. Установить библиотеку husky

```bash
npm install -g husky
```

2. Добавить хук husk в файл `package.json` в конец:

```js
"devDependencies": {
  ...
},
"husky": {
    "hooks": {
      "pre-commit": "pretty-quick --staged",
      "pre-push": "ng lint && ng test --browsers ChromiumNoSandbox"
    }
  }

```

Здесь я разделил команды: нет необходимости проверять тесты фронтенда на каждый коммит, но мы не дадим залить изменения в удаленный репозиторий, пока тесты не будут "зелеными".

# Итог

После того, как сделаны описанные в статье шаги, я получаю проект, который "защищает сам себя" от невалидного кода. Понятное дело, что одной проверкой синтаксиса и стайл-гайда не уберечь продукт от багов, однако даже эти незначительные вещи помогают в достижении большего качества кода и позволяют обсуждать архитектурные решения на код-ревью, а не вопросы форматирования или "табы или пробелы".

# Полезные ссылки


