# Авторизация пользователя в Web с помощью VK ID
  В данной статье мы рассмотрим, каким образом можно реализовать авторизацию пользователя 
с помощью VK ID. 
## Начало работы
  Для установки VK SDK необходимо воспользоваться менеджером пакетов.
Для этого в консоли выполните одну из команд, показанную ниже, в зависимости от используемого.

| Менеджер пакетов | Команда |
| ---------- | ------- |
|npm | ```npm install @vkontakte/superappkit ``` |
|yarn | ```yarn add @vkontakte/superappkit```  |

Также необходимо импортировать модуль **Connect** для настройки.
Для этого в консоли необходимо выполнить команду:
```
import { Connect } from '@vkontakte/superappkit';
```
## Настройка VK ID
  Возможность авторизации через VK ID в приложении будет доступна после настройки способа авторизации.
Для этого имеется несколько методов, вызываемых из модуля **Connect**.
Основным способом авторизации является метод **Connect.redirectAuth**.
```
// Connect.redirectAuth — Основной способ авторизации.
// VK ID в рамках текущей страницы откроет авторизацию.
// После успешной авторизации будет редирект на адрес, указанный в
параметре (url).
// url должен иметь схему https. При использовании http будет бесконечная
загрузка
// В url в query-string передастся параметр payload с данными авторизации,
// формат данных в payload - VKSilentTokenPayload.
// Также если передать параметр state, то после успешной авторизации в url
// в query-string будет передан параметр state имеющий такое же значение
// как и переданное в Connect.redirectAuth метод
// Если передать параметр screen со значением phone
// то мы всегда будем показывать экран с вводом номера телефона, 
// даже если нашли аккаунт пользователя
Connect.redirectAuth({ url: string, state?: string, screen?: 'phone' });
Connect.redirectAuth({ ..., action?: AuthAction });

```
В ответ VK SDK передаст результат авторизации, данные пользователя (идентификатор, первую букву фамилии, имя, аватар и маскированный номер телефона) и Silent token (параметр token).
Ответ будет выглядеть следующим образом:
```
payload:
{
auth: number;
token: string;
ttl: number;
type: string;
user: {
id: number;
first_name: string;
last_name: string;
avatar: string;
phone: string;
};
uuid: string;
oauthProvider?: string;
external_user?: ExternalUser;
};
```
Описание параметров ответа находятся в разделе "Описание типов".


  Для быстрой авторизации и размещения его окна на странице используется метод **Connect.oneTapAuth** и его аналоги: **Connect.floatingOneTapAuth** и **Connect.buttonOneTapAuth**.
Метод **Connect.oneTapAuth** в первое значение аргумента AuthButtonType принимает два значения *floating* и *button*. При значении *floating* окно авторизации размещается в правом верхнем углу 
в зафиксированном положении, а при значении аргумента *button* окно с кнопкой можно разместить в любом месте. Второй аргумент options представляет собой объект типа AuthButtonParams с
обязательным полем callback в который нужно передать функцию обработчик разных событий приходящих из SDK (см. ConnectEvents тип). В методе **Connect.buttonOneTapAuth** во втором
параметре доступны 2 свойства для управления отображения элементов.

Пример использования метода **Connect.floatingOneTapAuth**:
```
// Connect.floatingOneTapAuth - отображение окна быстрой авторизации в
правом
// верхнем углу экрана в виде position: fixed элемента.
// Аналог метода Connect.oneTapAuth('floating', ...)
Connect.floatingOneTapAuth({
callback: function(evt: any) {
switch (type) {
case ConnectEvents.OneTapAuthEventsSDK.LOGIN_SUCCESS:
if (evt.payload.auth) {
return onAuthUser(evt);
} else {
console.error(evt);
}
break;
// Для этих событий нужно открыть полноценный VK ID чтобы
// пользователь дорегистрировался или свалидировал телефон
case ConnectEvents.OneTapAuthEventsSDK.FULL_AUTH_NEEDED:
case ConnectEvents.OneTapAuthEventsSDK.PHONE_VALIDATION_NEEDED:
return Connect.redirectAuth({ screen: 'phone', url: 'https://...'
});
}
},
});
// Скрыть окно можно с помощью метода VKOneTapAuthResult.destroy
const oneTapObj = Connect.floatingOneTapAuth(...);

oneTapObj.destroy();
```

Использования метода **Connect.buttonOneTapAuth**:
```
// Connect.buttonOneTapAuth - окно с кнопкой быстрой авторизации, которое
можно
// вставить в любое место разметки. Во втором параметре доступно ещё 2
свойства:
// container - html элементы в который будет вставлено окно с кнопкой, и
options -
// ButtonOneTapAuthOptions, объект в этом параметре управляет показом
элементов
// внутри окна с кнопкой. Аналог метода - Connect.oneTapAuth('button',
...)
const buttonOneTap = Connect.buttonOneTapAuth({
callback: function(evt: any) {
const type = evt.type;
if (!type) {
return;
}
switch (type) {
...
case ConnectEvents.OneTapAuthEventsSDK.LOGIN_SUCCESS:
return onAuthUser(evt);
// Для событий PHONE_VALIDATION_NEEDED и FULL_AUTH_NEEDED
// нужно открыть полноценный VK ID чтобы пользователь
// дорегистрировался или свалидировал телефон
case ConnectEvents.OneTapAuthEventsSDK.FULL_AUTH_NEEDED:
case ConnectEvents.OneTapAuthEventsSDK.PHONE_VALIDATION_NEEDED:
case ConnectEvents.ButtonOneTapAuthEventsSDK.SHOW_LOGIN:
return Connect.redirectAuth({ screen: 'phone', url: 'https://...'
});
case ConnectEvents.ButtonOneTapAuthEventsSDK.SHOW_LOGIN_OPTIONS:
// Параметр screen: phone позволяет сразу открыть окно ввода телефона
// в VK ID
return Connect.redirectAuth({ screen: 'phone', source: type
}).then(onAuthUser, () => alert('Ошибка!'));
}
return;
},
options: {
showAlternativeLogin: true,
showAgreements: false,
showAgreementsDialog: true,
displayMode: 'default',
},
});
if (buttonOneTap) {
document.getElementById('someHtmlElementId').appendChild(buttonOneTap.getFrame());
}
```
Метод **Connect.userDataPolicy** необходим для отображения модального попапа (содержимое на странице будет недоступно для пользователя до тех пор, пока он не будет явно взаимодействовать с оверлеем).
Пример использования метода **Connect.userDataPolicy**:
```
// Connect.userDataPolicy - отображение модального попапа с политиками и
// передаваемыми данными
Connect.userDataPolicy(...)
.show()
.then(() => {
console.log('Policy was accepted');
});
```
## Доступные методы API

```Connect.redirectAuth(params: RedirectAuthParams): void;```

```Connect.oneTapAuth(oneTapAuthType: AuthButtonType, params: AuthButtonParams): VKOneTapAuthResult | null;```

```Connect.buttonOneTapAuth(params: ButtonOneTapAuthParams): VKOneTapAuthButtonResult | null;```

```Connect.floatingOneTapAuth(params: FloatingOneTapAuthParams): VKOneTapAuthResult | null;```

```Connect.userDataPolicy(uuid: string): VKDataPolicyResult```
## Описание типов 
Объект VKSilentAuthPayload
| Параметр | Тип | Описание |
| ---------- | ------- | ------- |
|auth|number |Признак активной сессии с ВКонтакте на устройстве пользователя.Возможные значения: 0 — сессии нет и 1 — сессия есть|
|token|string |Токен для совершения действий от лица пользователя. В данном случае Silent token, который необходимо обменять на Access token|
|ttl|string |Токен для совершения действий от лица пользователя. В данном случае Silent token, который необходимо обменять на Access token |
|type|string |Токен для совершения действий от лица пользователя. В данном случае Silent token, который необходимо обменять на Access token |
|user|string |Токен для совершения действий от лица пользователя. В данном случае Silent token, который необходимо обменять на Access token |
|id|string |Токен для совершения действий от лица пользователя. В данном случае Silent token, который необходимо обменять на Access token |
|first_name|string |Токен для совершения действий от лица пользователя. В данном случае Silent token, который необходимо обменять на Access token |
|last_name|string |Токен для совершения действий от лица пользователя. В данном случае Silent token, который необходимо обменять на Access token |
|avatar|string |Токен для совершения действий от лица пользователя. В данном случае Silent token, который необходимо обменять на Access token |\
|phone|string |Токен для совершения действий от лица пользователя. В данном случае Silent token, который необходимо обменять на Access token |
|uuid|string |Токен для совершения действий от лица пользователя. В данном случае Silent token, который необходимо обменять на Access token |
|oauthProvider?|string |Токен для совершения действий от лица пользователя. В данном случае Silent token, который необходимо обменять на Access token |
|external_user?|string |Токен для совершения действий от лица пользователя. В данном случае Silent token, который необходимо обменять на Access token |

Объект VKAuthSuccessResult
| Параметр | Тип | Описание |
| ---------- | ------- | ------- |
|auth|number |Признак активной сессии с ВКонтакте на устройстве пользователя.Возможные значения: 0 — сессии нет и 1 — сессия есть|
|token|string |Токен для совершения действий от лица пользователя. В данном случае Silent token, который необходимо обменять на Access token|

Объект ExternalUser
| Параметр | Тип | Описание |
| ---------- | ------- | ------- |
|auth|number |Признак активной сессии с ВКонтакте на устройстве пользователя.Возможные значения: 0 — сессии нет и 1 — сессия есть|
|token|string |Токен для совершения действий от лица пользователя. В данном случае Silent token, который необходимо обменять на Access token|

Объект ConnectAuthError
| Параметр | Тип | Описание |
| ---------- | ------- | ------- |
|auth|number |Признак активной сессии с ВКонтакте на устройстве пользователя.Возможные значения: 0 — сессии нет и 1 — сессия есть|
|token|string |Токен для совершения действий от лица пользователя. В данном случае Silent token, который необходимо обменять на Access token|

Объект RedirectAuthParams
| Параметр | Тип | Описание |
| ---------- | ------- | ------- |
|auth|number |Признак активной сессии с ВКонтакте на устройстве пользователя.Возможные значения: 0 — сессии нет и 1 — сессия есть|
|token|string |Токен для совершения действий от лица пользователя. В данном случае Silent token, который необходимо обменять на Access token|

