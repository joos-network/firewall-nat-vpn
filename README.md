# firewall-nat-vpn
Firewall, NAT, VPN

# Firewall

**Firewall** - дополнительный слой защиты между вами и проблемами

**Задача Firewall** - фильтрация проходящего через него трафика на основе определенных ранее правил. Это Защита сетей или отдельных хостов от атак, направленных извне внутрь защищаемой сети

**Netfilter** - межсетевой экран, встроенный в ядро Linux, начиная с версии 2.4

**iptables** - утилита Netfilter - Можно создавать и изменять правила, которые фильтруют трафик - является полноценным инструментом позволяющим настроить фаерволл

**firewalld** - Аналог iptables - В операционных системах CentOS, Fedora, OpenSUSE, Red Hat Enterprise Linux, SUSE Linux Enterprise

Архитектура Netfilter
- Подразумевает прохождение пакетов через цепочки правил
- Каждое правило содержит различные критерии и действие или переход, выполняющиеся в случае полного соответствия пакета критериям 
- Отсутствие критериев применяет правило ко всем проходящим через него пакетам

Cостав правила iptables
- Условие - Логическое выражение на основании которого происходит анализ свойств пакета / соединения и которое определяет попадание пакета / соединения под текущее правило
- Действие - Выполняется в случае соответствия пакета / соединения текущему правилу
- Счетчик - Учитывает количество пакетов попавших под условие текущего правила

**Цепочки iptables** - упорядоченная последовательность правил
- Пользовательская цепочка - Создаётся пользователем и используется только в пределах своей таблицы
- Базовая цепочка - Создаётся по умолчанию при создании таблицы и в отличии от пользовательской обладает действием по умолчанию

**Базовые цепочки iptables** - PREROUTING, INPUT, FORWARD, OUTPUT, POSTROUTING

**Таблица iptables** - совокупность базовых и пользовательских цепочек, имеющих общее назначение

**Conntrack (англ. отслеживание соединения)** - специальная подсистема, отслеживающая состояния соединений и позволяющая использовать эту информацию при принятии решений о судьбе отдельных пакетов

Состояния соединений
- NEW - пакет является первым в соединении
- ESTABLISHED - пакет относится к уже установленному соединению
- RELATED - пакет открывает новое соединение, логически связанное с уже установленными
- INVALID - установить принадлежность пакета не удалось
- UNTRACKED - отслеживание состояния соединения для данного пакета было отключено

**Host** - любое устройство подключенное к сети TCP/IP, принимающее или создающее подключения

**Localhost** - с помощью специального сетевого интерфейса «внутренней петли» (loopback) позволяет создавать сети, состоящие из одного компьютера

Цепочки iptables:
- Цепочка PREROUTING - Предмаршрутная обработка - Обрабатываются все входящие пакеты - Содержатся правила, обработка которых идет до решения о дальнейшей маршрутизации пакета (локальному процессу или другой машине в сети)
- - Для управления отслеживанием (таблица raw): отменить, настроить, ограничить отслеживание и т.д
- - Если необходимо модифицировать пакет до маршрутизации (mangle), например изменить поле TOS (IPv4), DSCP, TTL. Также можно сделать маркировку пакета или соединения
- - Для изменения адреса получателя в таблице nat: как IP-адреса через (Destination Network Address Translation) так и порта (с помощью действия REDIRECT) (Скрытие порта приложения с помощью таблицы nat (REDIRECT с внешнего порта 12345 на 22 в локальной сети))
- Цепочка INPUT - Входящие пакеты - Обрабатываются входящие пакеты, адресованные локальному процессу
- - Изменение заголовка пакета, прежде чем он попадет к локальному процессу (таблица mangle)
- - Фильтрация входящего трафика (таблица filter)
- - Передача специфичным системам принудительного контроля доступа (security) Данная таблица появляется только с использованием возможностей SELinux
- - Иногда необходимо обработать два идентичных потока из разных зон, когда получателем выступает машина с фаерволом. В таких случаях в цепочке INPUT используется таблица nat
- Цепочка FORWARD - Пересылка пакетов - Обрабатываются входящие пакеты, которые пересылаются дальше
- - В исключительных случаях вносить изменение в заголовок транзитного пакета
- - Фильтрация трафика, идущего в обоих направлениях (в локальную и внешнюю сеть)
- Цепочка OUTPUT - Исходящие пакеты - Предназначена для пакетов, исходящих от внутренних процессов
- - Управления отслеживанием (таблица raw), как то: задать зону conntrack для пакета, отменить, настроить, ограничить отслеживание и т.д.
- - Внесение изменений в заголовок исходящего пакета (таблица mangle)
- - Повторения в случае необходимости подмены адресов (IP, порт TCP) для локально созданных пакетов (таблица nat)
- - Обычной (таблица filter) и усиленной (таблица security) фильтрации исходящих пакетов
- Цепочка POSTROUTING - Постмаршрутизация - Происходит окончательная обработка исходящих пакетов
- - Внесения изменения в заголовок исходящего или транзитного пакета/сегмента уже после того, как принято последнее решение о маршрутизации (таблица mangle)
- - Замены адреса отправителя (Source Network Address Translation), проводить операции маскарадинга (таблица nat)

**Утилита netstat** - позволяет смотреть состояния соединений, таблиц маршрутизации, чисто сетевых интерфейсов и статистику по протоколам

- **netstat -l** - Посмотреть все сокеты с состоянием LISTEN
- **netstat -s** - Узнать статистику для каждого протокола

Шаблон работы с iptables
- iptables [-t table] command [match] [target/jump]
- -t – указывает на таблицу (raw, mangle, nat, security), по умолчанию без указания параметра выбирается таблица filter
- [match] – задает критерии проверки, по которым определяется подпадает ли пакет под действие этого правила или нет
- [target] – указывает, какое действие должно быть выполнено при условии выполнения критериев в правиле

При начальной настройке всегда нужно задавать политику обработки пакетов по умолчанию для каждой цепочки
- sudo iptables -P INPUT DROP
- sudo iptables -P FORWARD DROP

Команда для просмотра таблиц iptables
- sudo iptables -nvL -t raw
- sudo iptables -nvL -t mangle
- sudo iptables -nvL -t nat
- sudo iptables -nvL -t filter

Пример добавления правила iptables
- Используем команду:
- sudo iptables -A INPUT -p tcp --dport 22 -m state --state NEW,ESTABLISHED -s 192.168.0.0/24 -j ACCEPT
- -A INPUT – (append, добавить) указывает цепочку (например, INPUT ) для добавления правила
- -p tcp – указываем сетевой протокол (например, tcp или udp )
- --dport 22 – порт назначения пакетов
- -m state – критерий, свойство пакета, которое мы хотим сопоставить (например, state )
- --state NEW, ESTABLISHED – состояние(я) пакета для соответствия
- -s 192.168.0.0/24 – (source, источник) IP-адрес и маска источника, из которого исходят пакеты
- -j ACCEPT – цель или что делать с пакетами (например, ACCEPT, DROP, REJECT и т. д.)

**Ebtables** - средство для фильтрации пакетов для программных мостов Linux, работает преимущественно на втором (канальном) уровне модели OSI. Ebtables предназначена для фильтрации трафика в bridge

Чтобы отбросить трафик от конкретного MAC адреса в ebtables необходима следующая команда:
- ebtables -A INPUT -s 08:00:27:47:88:CE -j DROP
Вариант, который мы использовали в iptables:
- sudo iptables -A INPUT -m mac --mac-source 08:00:27:47:88:CE -j DROP


# NAT

**Частные IPv4-адреса** не являются уникальными и могут использоваться во внутренней сети

curl ifconfig.me - узнать белый ip

whois $(curl ifconfig.me) - узнать информацию whois

Блоками частных адресов являются
- 10.0.0.0/8 от 10.0.0.0 до 10.255.255.255
- 172.16.0.0/12 от 172.16.0.0 до 172.31.255.255
- 192.168.0.0/16 от 192.168.0.0 до 192.168.255.255

Способы выхода в интернет с приватных адресов
- Использование сервера PROXY
- Подмена адресов NAT

**Network Address Translation** -  Преобразование сетевых адресов

**NAT** - специальный механизм, реализованный в сетях TCP/IP, который позволяет изменять IP-адреса и/или номера портов TCP/UDP в пересылаемых пакетах

Виды NAT
- Static NAT / One to one NAT
- Dynamic NAT
- Source NAT / NAT Overload / Masquerade / Many to one
- Destination NAT (PAT)

Преимущества NAT
- Под одним внешним IPv4 адресом может сидеть в глобальной паутине множество пользователей одновременно
-  Скрывает ваш настоящий внутренний IP в частной сети и показывает лишь внешний. Так, все устройства из вне видят только ваш общедоступный IP
- В определенной степени выполняет функции фаервола – если на устройство с NAT извне приходит пакет, который не ожидался — то он категорически не будет допущен

Недостатки NAT
- Невысокая скорость передачи данных для протоколов реального времени, например, для VoIP. Когда NAT переделывает заголовки пакета — происходят задержки
- Проблемы с идентификацией – под одним IP может находится сразу несколько человек.
- Сервис может заблокировать внешний IP-адрес из-за злоумышленника (который, например, подбирает пароли), а работать перестанет у всех, кто закрывается этим адресом

**Static NAT** -  сопоставление локального IP-адреса с глобальным IP- адресом на основании один к одному

**Static NAT** - используется при необходимости «опубликовать» внутренний сервер компании в интернет, причём не один, а все сервисы сразу. Несмотря на то, что у сервера «серый» адрес, он
полноценно отвечает на запросы извне (по другому, «белому» адресу)

**Dynamic NAT** - динамическая адресная трансляция, в которой адреса сопоставляются по принципу «многие ко многим». Реальные адреса выдаются динамически каждому нуждающемуся пользователю во внутренней сети, а не одному определенному узлу. Dynamic NAT – используется редко

**Механизм Source NAT (SNAT)** - позволяет изменить исходный IP-адрес сетевого пакета на другой IP-адрес, а также увеличить безопасность и сохранить конфиденциальность, поскольку маскируются и скрываются частные IP-адреса устройств

**Source NAT** -  используются для выхода в интернет группы компьютеров с внутренними адресами через один внешний адрес. Снаружи на внешний адрес пропускаются только пакеты, содержащиеся в таблице трансляций

**NAT Masquerading** -  тип трансляции сетевого адреса, при которой внешний адрес отправителя подставляется динамически, в зависимости от назначенного провайдером адреса

**NAT Masquerading** -  используются для выхода в интернет группы компьютеров с внутренними адресами через один внешний адрес. IP-адрес, в который происходит подмена, должен быть прописан на интерфейсе

**Destination NAT / PAT (DNAT)** - технология трансляции сетевого адреса в зависимости от TCP/UDP-порта получателя

**Destination NAT / PAT** -  используются для публикации сервиса (port), находящегося внутри сети, для внешних пользователей

NAT в Linux реализуется с помощью **iptables**

Для начала необходимо разрешить пересылку пакетов с одного интерфейса на другой (по умолчанию в Linux это отключено)
```
# до перезагрузки
sudo sysctl -w net.ipv4.ip_forward=1
# на постоянной основе
/etc/sysctl.conf =>
net.ipv4.ip_forward = 1
$ sudo sysctl -p /etc/sysctl.conf
```

Создадим правило в iptables, разрешающее передачу пакетов между внутренним (eth1) и внешним (eth0) интерфейсом и разрешим передавать между интерфейсами пакеты, относящиеся к уже установленным соединениям
```
iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT
iptables -A FORWARD -m state --state RELATED,ESTABLISHED -j ACCEPT
```
- -A – add
- -i / -o – input/output
- -m – использовать доп.модуль
- -j – действие

Включим SNAT:
```
iptables -t nat -A POSTROUTING -s 10.2.0.0/24 -o eth1 -j SNAT --to-source 84.201.168.122
```
- -A – add;
- -t – таблица;
- -s – source (необязательно);
- -j – действие;
- --to-source – должен быть адресом на интерфейсе, с которого планируется выпускать во внешнюю сеть IP пакеты

```
iptables -t nat -L -n -v
```

Команды DNAT в Linux
 - Сначала разрешим передачу пакетов с внешнего интерфейса (eth0) на внутренний (eth1) интерфейс:
- При необходимости можно отдельным правилом запретить подключение через NAT для отдельных IP адресов или подсетей
```
iptables -A FORWARD -i eth0 -o eth1 -j ACCEPT
iptables -I FORWARD 1 -o eth1 -s 167.71.67.136 -j DROP
```
- Теперь перенаправим все соединения на порт 80 интерфейса внешней сети (eth0) на IP адрес веб сервера внутренней сети web-server01
- И все соединения на порт 13389 перенаправлять на порт 3389 сервера внутренней сети (в целях безопасности)
```
iptables -t nat -A PREROUTING -p tcp --dport 80 -i eth0 -j DNAT --to-destination 10.2.0.11
iptables -t nat -A PREROUTING -p tcp --dport 13389 -i eth0 -j DNAT --to-destination 10.2.0.12:3389
```

# VPN

**Virtual Private Network** - Виртуальная частная сеть - механизм, позволяющий настроить подключение устройств через сети общего доступа, так, как если бы они находились в одной (частной) сети

Зачем нужен VPN
- Полный контроль сети (private) - Задать жесткое соответствие между IP адресом и пользователем/устройством
- Шифрование  - Передавать данные в зашифрованном виде, повышая безопасность передаваемых данных
- Единая адресация, разграничение доступа - Разграничить доступ квнутренним ресурсам основываясь только на L3 адресах
- Контроль действий сотрудников - На основе адресов можно вестистатистику запросов с привязкой к пользователю
- Сокрытие / маскировка реального IP-адреса - Анонимность запросов, обезличивание как защита от неправомерных действий через социальную инженерию
- Доступ к заблокированным ресурсам - Обеспечить доступ к недоступнымпо региональному признаку ресурсу

Виды VPN
- Точка - точка (Point-to-Point)
- Точка - Сеть (Remote Access)
- Сеть - сеть (Site-to-Site)
- VPN-сервис в браузере

**VPN Point-to-Point** (p2p, точка-точка) - используются для соединения между собой двух устройств

**VPN-туннель** - зашифрованное подключение между двумя точками, VPN клиентом и VPN сервером, через общедоступные сети. При взаимодействии все реальные промежуточные узлы будут скрыты

**VPN Remote access** - пример - Подключение с помощью веб-браузера к внутреннему ресурсу компании, в нашем случае – веб серверу. Так часто реализуется подключение к Outlook Web Access (OWA) client

Clientless SSL VPN веб-портал позволяет предоставить доступ к внутренним веб-ресурсам, терминальным и SSH-серверам компании для удаленных или мобильных пользователей, используя защищенное HTTPS-соединение веб-браузера

Требования для Clientless SSL VPN
- Специальное оборудование, поддерживающее данный режим
- Настройка отдельного сервера

**Cisco Anyconnect** - Популярный корпоративный VPN клиент - VPN Подключение с помощью специального ПО (VPN клиента) к VPN серверу компании с использованием шифрования и авторизации

**VPN Site-to-Site** - используются для объединения удалённых офисов через публичную сеть интернет. Для конечных пользователей VPN выглядит прозрачным, при трассировке никаких «белых» IP- адресов никто не увидит

VPN протоколы
- PPTP - Point-to-Point Tunneling Protocol
- SSTP - Secure Socket Tunneling Protocol
- WireGuard 
- OpenVPN
- L2TP / IPSec - Layer 2 Tunneling Protocol / IP Security

**PPTP** -  туннельный протокол типа точка-точка, позволяющий компьютеру устанавливать защищённое соединение с сервером за счёт создания специального туннеля в стандартной, незащищённой сети
- Работает на L4 уровне, поверх TCP
- Предоставляет сервисы L2 уровня

**PPTP использует два соединения**
- Одно соединение для управления - Работает с использованием TCP, в котором порт сервера 1723
- Второе соединение для инкапсуляции данных - Работает с помощью протокола GRE, который является транспортным протоколом (то есть заменой TCP/UDP)

Особенности PPTP
- Несмотря на относительно высокую скорость, PPTP не слишком надежен: после обрыва соединения он не восстанавливается так же быстро, как, например, OpenVPN
- В настоящее время PPTP по существу устарел и Microsoft советует пользоваться другими VPN решениями, так как обладает серьёзными уязвимостями

**L2TP** - протокол туннелирования второго уровня, используется для поддержки виртуальных частных сетей
- Работает на L4 уровне, поверх UDP
- Предоставляет сервисы L2 уровня

**Протокол L2TP** при передачи данных по туннелю L2TP кадр L2TP помещается в дейтаграмму UDP и пересылается конечной точке

Какие особенности L2TP?
- L2TP / IPsec считается безопасным и не имеет серьезных выявленных проблем (гораздо безопаснее, чем PPTP)
- L2TP / IPsec может использовать шифрование 3DES или AES, хотя, учитывая, что 3DES в настоящее время считается слабым шифром, он используется редко
- У протокола L2TP иногда возникают проблемы из-за использования по умолчанию UDP-порта 500, который иногда блокируется брандмауэрами

**SSTP** - безопасный протокол туннелирования сокетов, проприетарный продукт от Microsoft
- Работает на Прикладном уровне L7
- Предоставляет сервисы L2 уровня

**Протокол SSTP** - отправляет трафик по SSL через TCP-порт 443

Особенности SSTP
- Как и PPTP не очень широко используется в VPN, но, в отличие от PPTP, у него не диагностированы серьезные проблемы с безопасностью
- SSTP удобен для использования в ограниченных сетевых ситуациях, например, если вам нужен VPN для Китая
- Несмотря на то, что SSTP также доступен и на Linux, RouterOS и SEIL, по большей части он все равно используется Windows- системами

**WireGuard** - коммуникационный протокол и бесплатное программное обеспечение с открытым исходным кодом, который реализует зашифрованные виртуальные частные сети
- Работает поверх L4 (UDP)
- Предоставляет сервисы L2 уровня

**WireGuard** - один из новейших VPN протоколов, распространяется по GNU GPL

Особенности WireGuard
- WireGuard не предоставляет выбор алгоритма или уровня шифрования. Благодаря этому:
- - Код сократился до 4 тысяч строк, что позволяет легче осуществлять аудит безопасности
- - Настройка осуществляется гораздо проще
- - Работает гораздо быстрее с приемлемым уровнем безопасности
- В Linux системах реализован в виде модуля ядра, что ополнительно величивает роизводительность
- Протокол может аботать на любом из ортов UDP, уществуют клиенты од все основные латформы: Windows, ac OS, Linux, Apple OS, Android

**OpenVPN** - универсальный протокол VPN с открытым исходным кодом, разработанный компанией OpenVPN Technologies
- Работает на L4 уровне, поверх TCP и UDP
- Предоставляет сервисы на уровне L2

**OpenVPN** самый популярный протокол VPN. Будучи открытым стандартом, он прошел не одну независимую экспертизу безопасности. OpenVPN использует библиотеку OpenSSL для шифрования и аутентификации, большинство VPN-сервисов создают свои приложения для работы с OpenVPN, которые можно использовать в разных операционных системах и устройствах

Особенности OpenVPN
- Для работы OpenVPN нужно специальное клиентское программное обеспечение
- OpenVPN является альтернативой IPsec тогда, когда провайдерблокирует некоторые протоколы VPN
- Протокол можетработать на любом из портов TCP и UDP и может использоваться на всех основных платформах через сторонние клиенты: Windows, Mac OS, Linux, Apple iOS, Android

**IPSec** -  стек сетевых протоколов для защищенной передачи данных через IP-сети

IPsec обеспечивает
- Аутентификацию
- Шифрование
- Проверку целостности передаваемых данных

Режимы IPsec
- Транспортный режим - Работает поверх протокола IP и шифрует содержимое IP-пакета (payload) - Адреса отправителя и получателя не шифруются, поэтому можно проанализировать адреса и объем переданной информации
- Туннельный режим Создает новый IP-пакет, полностью шифруя исходный - Использование туннельного режима сильно затрудняет анализ перехваченного трафика

- Транспортный режим - Чаще всего используется для соединения между хостами
- Туннельный режим - Чаще всего используется для передачи данных через Интернет

**Туннелирование (tunneling)** -  метод, используемый для передачи полезной нагрузки (кадра или пакета) одного протокола с использованием межсетевой инфраструктуры другого протокола

**Tunnel** - Вкладываются данные этого же или более нижних сетевых уровней

**Security Association** - базовое понятие IPsec. Включает в себя информацию о криптографических протоколах и алгоритмах, лючах шифрования, определяет какие данные будут проходить через туннель

Алгоритм работы IPsec
- Установка и поддержка VPN туннеля происходит в два этапа
- - В фазе один, узлы договариваются о:
- - Методе идентификации 
- - Алгоритме шифрования
- - Хэш алгоритме 
- - Группе Diffie Hellman
- - Также происходит взаимная идентификация
- Если шаги в первой фазе завершились успешно, то создаётся SA первой Фазы (Phase 1 SA или IKE SA) и процесс переходит к фазе два
- - В этой фазе генерируются ключи и узлы договариваются об используемой политике
- Если вторая фазы выполняется успешно, то создается Phase 2 SA или IPSec SA. На этом установка туннеля считается завершенной
