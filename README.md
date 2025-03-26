### Реализация авторизации пользователей в Yii2


#### Шаг 1: Настройка модели User для авторизации

1. Откройте `models/User.php`
2. Перенесите код из ранее переименованной модели (например, `UserOld`)
3. Добавьте к классу User `implements IdentityInterface`

```php
use yii\web\IdentityInterface;

class User extends \yii\db\ActiveRecord implements IdentityInterface
{
    // ... существующий код ...

    public static function findIdentity($id)
    {
        return static::findOne($id);
    }

    public static function findIdentityByAccessToken($token, $type = null)
    {
        return null; // Не используем токены
    }

    public function getId()
    {
        return $this->id;
    }

    public function getAuthKey()
    {
        return null; // Не используем authKey
    }

    public function validateAuthKey($authKey)
    {
        return false; // Не используем authKey
    }

    public static function findByUsername($username)
    {
        return static::findOne(['username' => $username]);
    }

    public function validatePassword($password)
    {
        return Yii::$app->security->validatePassword($password, $this->password);
    }
}
```


---


#### Шаг 2: Создание формы входа

1. Создаем `models/LoginForm.php`:

```php
namespace app\models;

use yii\base\Model;

class LoginForm extends Model
{
    public $username;
    public $password;
    public $rememberMe = true;
    private $_user = false;

    public function rules()
    {
        return [
            [['username', 'password'], 'required', 'message' => 'Заполните поле'],
            ['rememberMe', 'boolean'],
            ['password', 'validatePassword'],
        ];
    }

    public function validatePassword($attribute)
    {
        if (!$this->hasErrors()) {
            $user = $this->getUser();
            if (!$user || !$user->validatePassword($this->password)) {
                $this->addError($attribute, 'Неверное имя пользователя или пароль.');
            }
        }
    }

    public function login()
    {
        if ($this->validate()) {
            return Yii::$app->user->login(
                $this->getUser(),
                $this->rememberMe ? 3600*24*30 : 0
            );
        }
        return false;
    }

    protected function getUser()
    {
        if ($this->_user === false) {
            $this->_user = User::findByUsername($this->username);
        }
        return $this->_user;
    }
}
```

*картинка с (модель LoginForm)*

---

#### Шаг 3: Создание контроллера для авторизации

1. Модифицируем `controllers/SiteController.php`:

```php
public function actionLogin()
{
    if (!Yii::$app->user->isGuest) {
        return $this->goHome();
    }

    $model = new LoginForm();
    
    if ($model->load(Yii::$app->request->post()) && $model->login()) {
        return $this->goBack();
    }

    $model->password = '';
    return $this->render('login', [
        'model' => $model,
    ]);
}

public function actionLogout()
{
    Yii::$app->user->logout();
    return $this->goHome();
}
```

*картинка с (действия login и logout в SiteController)*

---

#### Шаг 4: Создание представления для входа

1. Создаем `views/site/login.php`:

```php
<?php
use yii\helpers\Html;
use yii\widgets\ActiveForm;

$this->title = 'Вход';
?>
<h1><?= Html::encode($this->title) ?></h1>

<?php $form = ActiveForm::begin(); ?>
    <?= $form->field($model, 'username')->textInput(['autofocus' => true]) ?>
    <?= $form->field($model, 'password')->passwordInput() ?>
    <?= $form->field($model, 'rememberMe')->checkbox() ?>
    
    <div class="form-group">
        <?= Html::submitButton('Войти', ['class' => 'btn btn-primary']) ?>
    </div>
<?php ActiveForm::end(); ?>

<p>Ещё не зарегистрированы? <?= Html::a('Регистрация', ['user/register']) ?></p>
```

*картинка с (представление формы входа)*

---

#### Шаг 5: Настройка компонента user

1. В `config/web.php` добавляем:

```php
'components' => [
    'user' => [
        'identityClass' => 'app\models\User',
        'enableAutoLogin' => true,
        'loginUrl' => ['site/login'],
    ],
    // ... другие компоненты ...
],
```

*картинка с (настройка компонента user)*

---

#### Шаг 6: Проверка авторизации в других контроллерах

Для ограничения доступа к определенным действиям используем behaviors:

```php
public function behaviors()
{
    return [
        'access' => [
            'class' => \yii\filters\AccessControl::className(),
            'rules' => [
                [
                    'allow' => true,
                    'actions' => ['login', 'register'],
                    'roles' => ['?'], // Только для гостей
                ],
                [
                    'allow' => true,
                    'roles' => ['@'], // Только для авторизованных
                ],
            ],
        ],
    ];
}
```

*картинка с (настройка доступа в контроллере)*

---

#### Шаг 7: Проверка работы авторизации

1. Перейдите по адресу `http://localhost/cleaning_portal/web/site/login`
2. Введите данные зарегистрированного пользователя
3. Проверьте:
   - После успешного входа происходит перенаправление
   - В верхнем меню отображается имя пользователя
   - Кнопка "Выйти" работает корректно
   - Неавторизованные пользователи не могут создать заявку

*картинка с (успешная авторизация пользователя)*

---

### Итог

Мы реализовали:
1. Адаптированную модель User для авторизации
2. Форму входа с валидацией
3. Контроллер для обработки входа/выхода
4. Настроили компонент user
5. Добавили ограничения доступа

Теперь система имеет полноценную авторизацию, интегрированную с нашей базой данных. Для административной панели потребуется дополнительная настройка RBAC, которую мы рассмотрим в следующей лекции.
