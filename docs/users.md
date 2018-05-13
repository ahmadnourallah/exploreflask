# أساليب للتعامل مع المُستخدمين

<img src='../images/users.png'/>

أحد أكثر الأمور شيوعاً التي تحتاج تطبيقات الويب العصريّة إلى القيام بها هي التعامل مع المُستخدمين. فالتطبيق الذي يوفر أبسط مزايا التعامل مع حسابات المُستخدمين يحتاج للتعامل مع عشرات الأشياء في الخلفيّة، كإنشاء الحساب، وتأكيد البريد الإلكتروني، وتخزين كلمات المرور بشكل آمن، وإعادة تعيين كلمات المرور بشكل آمن، والمصادقة (authentication)، وغيرها من الأمور. وبما أنَّ العديد من المشاكل الأمنيّة تطوف في هذه الناحيّة، فمن الأفضل عليك استخدام الأساليب المعياريّة والشائعة في مشاريعك.

<blockquote>
<b>ملاحظة</b><br/>
أفترض في هذا القسم أنَّك تستخدم إضافة SQLAlchemy للتعامل مع قواعد البيانات وإضافة WTForms للتعامل مع الاستمارات. ستضطر إلى ملائمة الأساليب المذكورة هنا مع أدواتك إن لم تكن تستخدم ما سلف ذكره.
</blockquote>

## تأكيد البريد الإلكتروني

عادةً ما تقوم تطبيقات الويب بتأكيد البريد الإلكتروني المُدخَل من المُستخدمين الجدد للتأكد من أنَّه صالح وصحيح. فبعد التأكّد من صحة البريد، يمكننا إرسال روابط لإعادة تعيين كلمات المرور وغيرها من المعلومات الحساسة لمُستخدمينا براحة تامة وبدون القلق حول من يتلقى تلك الرسائل.

أحد أكثر الأساليب شيوعاً لإنجاز ذلك هو إرسال رابط لإعادة تعيين كلمة المرور يحوي جزئيّة فريدة، فعند الدخول إلى ذلك الروابط نتيقَّن من أنَّ بريد المُستخدِم صحيح. لنأخذ مثال على ذلك: مُستخدِم بريده john@gmail.com قام بإنشاء حساب في تطبيقنا. قمنا بتسجيله في قاعدة البيانات بنجاح، وأضفنا القيمة `False` إلى العمود `email_confirmed`، ومن ثم أرسلنا رسالةً إلى بريده الإلكتروني (john@gmail.com) تحوي رابطاً بجزئيّة فريدة. تكون هذه الجزئيّة مميزة وبقيمة مُبعثرة (أو تبدو كذلك)، كهذه: http://myapp.com/accounts/confirm/Q2hhZCBDYXRsZXR0IHJvY2tzIG15IHNvY2tz. سيضغط صاحب البريد (جون) عند استقباله للرسالة على الرابط المُرفَق ليُؤكِّد حسابه لدينا. سيستقبل التطبيق الجزئيّة الفريدة، ومنها سيعلم البريد الذي عليه تأكيده، وبعدها سيقوم بوضع القيمة `True` في العمود `email_confirmed` في حساب صاحب البريد.

ستتساءل، بالتأكيد، عن الكيفيّة التي سنعلم بها البريد الذي علينا تأكيده من تلك الجزئيّة. إحدى الطرق للقيام بذلك عبر حفظ الجزئيّة في قاعدة البيانات عند إنشائها وإعادة التأكّد منها عند تلقي طلب التأكيد. هذه الطريقة مُزعِجة وستجلب الكثير من الصداع. لحسن حظنا أنّنا لن نحتاج لها.

سنقوم بدلاً من ذلك بترميز البريد الإلكتروني بداخل الجزئيّة. ستحتوي الجزئيّة أيضاً على طابع زمني (timestamp) ليخبرنا بمدّة صلاحيّتها. سنستخدم مكتبة `itsdangerous` للقيام بذلك. تتيح لنا هذه المكتبة إمكانيّة إرسال بيانات حساسة إلى بيئات غير موثوقة (كإرسال رسالة تحوي رمز تفعيل إلى بريد إلكتروني غير مؤكّد، كما في حالتنا) بشكل آمن. سنستخدم الصنف `URLSafeTimedSerializer` هنا من هذه المكتبة.

```python
# ourapp/util/security.py

from itsdangerous import URLSafeTimedSerializer

from .. import app

ts = URLSafeTimedSerializer(app.config["SECRET_KEY"])
```

سنستخدم المُسلسِل (serializer) لتوليد رمز تفعيل مُميّز عندما نحصل على البريد الإلكتروني للمُستخدِم. سننفذ أسلوب بسيط لإنشاء حسابات المُستخدمين بهذه الطريقة.

```python
# ourapp/views.py

from flask import redirect, render_template, url_for

from . import app, db
from .forms import EmailPasswordForm
from .util import ts, send_email

@app.route('/accounts/create', methods=["GET", "POST"])
def create_account():
    form = EmailPasswordForm()
    if form.validate_on_submit():
        user = User(
            email = form.email.data,
            password = form.password.data
        )
        db.session.add(user)
        db.session.commit()

        # توجد هنا جزئيّة إرسال البريد الإلكتروني
        subject = "Confirm your email"

        token = ts.dumps(self.email, salt='email-confirm-key')

        confirm_url = url_for(
            'confirm_email',
            token=token,
            _external=True)

        html = render_template(
            'email/activate.html',
            confirm_url=confirm_url)

        # دعنا نفترض أنَّ هذه الدالة (send_eamil) تم تعريفها في الوحدة myapp/util.py
        send_email(user.email, subject, html)

        return redirect(url_for("index"))

    return render_template("accounts/create.html", form=form)
```

تقوم الدالة السابقة بإنشاء حساب المُستخدِم وإرسال رسالة إلى بريده الإلكتروني. سنستخدم في مثالنا قالب لتوليد رسالة التأكيد الإلكترونيّة (انظر أدناه).

```html
{# ourapp/templates/email/activate.html #}

Your account was successfully created. Please click the link below<br>
to confirm your email address and activate your account:

<p>
<a href="{{ confirm_url }}">{{ confirm_url }}</a>
</p>

<p>
--<br>
Questions? Comments? Email hello@myapp.com.
</p>
```

بقي الآن لدينا أن نقوم بكتابة دالة عرض لتستقبل طلبات التأكيد.

```python
# ourapp/views.py

@app.route('/confirm/<token>')
def confirm_email(token):
    try:
        email = ts.loads(token, salt="email-confirm-key", max_age=86400)
    except:
        abort(404)

    user = User.query.filter_by(email=email).first_or_404()

    user.email_confirmed = True

    db.session.add(user)
    db.session.commit()

    return redirect(url_for('signin'))
```

هذه الدالة مُجرَّد مثال بسيط. قمنا بدايةً بإضافة العبارة `try ... except` للتأكّد ممّا إذا كان رمز التفعيل المُستقبَل صالحاً. سيحتوي رمز التفعيل أيضاً على طابع زمني، ولهذا قمنا باستدعاء الدالة الصنفيّة `()ts.loads` لكي تُصدِر خطأً إذا كان الطابع الزمني المُستقبَل أكبر من الموضوع في المُعامِل `max_age`. قمنا بتعيين قيمة المُعامِل `max_age` في مثالنا إلى 86400 ثانية (أي 24 ساعة).

<blockquote>
<b>ملاحظة</b><br/>
يمكنك تنفيذ خاصيّة تحديث البريد الإلكتروني بطريقة مماثلة لتلك. كل ما عليك هو إرسال رابط التأكيد إلى البريد الإلكتروني الجديد وفيه رمز التفعيل ولكن هذه المرَّة يجب أن يحتوي على البريدان الإلكترونيان: القديم والجديد. وعند تلقي الطلب ستقوم بالتحقق ممّا إذا كان صحيحاً والتصرف تبعاً لذلك (تحديث البريد أو إرسال رسالة خطأ للمُستخدِم).
</blockquote>

## تخزين كلمات المرور

القاعدة رقم واحد في التعامل مع بيانات المُستخدمين هي تهشير (hash) كلمات مرورهم باستخدام خوارزميّة Bcrypt (أو خوارزميّة scrypt، ولكنني سأستخدم Bcrypt هنا) قبل تخزينها. فلا أحد يقوم بتخزين كلمات المرور كنص عادي. قيامك بهذا سيُحدِث مشاكل أمنيّة كبيرة وغير مُنصِفة بحق بيانات مُستخدميك. لا يوجد شيء شاق هنا، فكل الأمور الصعبة والمُعقدة قد اُختصِرَت ويُسِرَت لتُسهِل علينا الطريق. لا عذر لأن لا نتبع الممارسات المعياريّة بهذا الشأن.

<blockquote>
<b>ملاحظة</b><br/>
تعد منصة أواسب (OWASP) إحدى أكثر المنصات الموثوقة في المجالات المتعلقة بحمايّة تطبيقات الويب. احرص على إلقاء نظرة على <a href='https://www.owasp.org/index.php/Secure_Coding_Cheat_Sheet#Password_Storage'>توصياتهم بشأن كتابة كود آمن ونظيف (من المشاكل الأمنيّة)</a>.
</blockquote>

سنستخدم هنا إضافة Flask-Bcrypt التي تتيح لنا إمكانيّة استخدام مكتبة bcrypt الشهيرة في موقعنا. هذه الإضافة هي مُجرَّد غلاف (wrapper) لمكتبة `py-bcrypt`، ولكنها تقوم ببعض التكتيكات الإضافية وتتعامل مع بضعة أمور من المُزعِج القيام بها يدوياً (كالتأكُّد من الترميز النصي ﻷكواد الهاش قبل مقارنتها).

```python
# ourapp/__init__.py

from flask_bcrypt import Bcrypt

bcrypt = Bcrypt(app)
```

أحد الأسباب الذي يجعل خوارزميّة Bcrypt موصى بها بشدّة في تهشير البيانات أنَّها "مُتكيّفة مُستقبلياً"، بمعنى أنَّه يمكننا زيادة صعوبة التخمين على البيانات المُهشَّرة بواسطتها عبر زيادة القوّة الحاسوبيّة المُستخدمة في العمليّة. فكلّما زادت "الجولات" المُستخدمة لتهشير كلمة مرور كُلّما استغرقت عمليّة تخمينها وقتاً أطولاً. حيث إنْْ قمنا بعشرين جولة تهشيريّة، مثلاً، على كلمة مرور قبل خزنِها سيضظر المهاجِم إلى فك تهشير الكلمة عشرين مرَّة أيضاً.

تذكر أنَّ وقت إنهاء العمليّة سيطول  مع زيادة الجولات المُستخدمة. مما يعني أنَّه عليك الموازنة ما بين الأمان وقابليّة الاستخدام عند اختيارك لعدد الجولات الذي سيُجرى. يعتمد عدد الجولات الذي يُمكِن إجرائه في مقدار مُعين من الوقت على الموارد الحاسوبيّة للخادِم التي يمكن للموقع استخدامها. عليك محاولة تجربة عدّة أرقام واختيار رقم يستغرق مابين 0.25 و 0.5 تقريباً لتهشير كلمة المرور الواحدة. عليك المحاولة أيضاً ألا يقل ذلك الرقم عن 12.

جرّب البريمج البايثوني البسيط التالي لاختيار الوقت المناسب لك عند تهشير كلمات المرور:

```python
# benchmark.py

from flask_bcrypt import generate_password_hash

# عدِّل عدد الجولات (في المُعطى الثاني) لحين أن يصل الوقت المُستغرَق
# بين 0.25 و 0.5 ثانيّة في عمليّة التهشير الواحدة.
generate_password_hash('password1', 12)
```

والآن قم بتشغيل البريمج عبر أداة `time` لأنظمة يونكس لقياس الوقت المُستغرَق لتنفيذ العمليّة:

```
$ time python test.py

real    0m0.496s
user    0m0.464s
sys     0m0.024s
```

جربت تلك العمليّة على خادم صغير لدي ووجدت أنَّ 12 جولة تستغرق وقت مناسب ولهذا سأضبط مثالنا ليستخدم عدد الجولات ذاك.

```python
# config.py

BCRYPT_LOG_ROUNDS = 12
```

والآن يمكننا كتابة جزئيّة عمليّة التهشير بعد إنهاء ضبط إعدادات الإضافة. يمكننا وضع هذه الجزئيّة يدوياً في دالة العرض التي تستقبل طلب التسجيل من استمارة إنشاء الحساب، ولكننا حينها سنضطر إلى إعادة العمليّة يدوياً في دالة استرجاع كلمة المرور وإعادة تعيينها، ولذلك سأقوم بهذا بطريقة مُختصرة وأكثر ديناميكيّة بحيث أنّنا لن نعيدها إطلاقاً في جميع عمليات تخزين كلمات المرور. سأقوم باستخدم خاصيّة **الضابِط** (setter) التي توفرها إضافة SQLAlchemy، حيث أنَّنا سنستخدمها لتقوم تلقائياً بتهشير كلمات المرور باستخدام Bcrypt قبل تخزينها عند استخدام السطر `'user.password = 'password1` في أي مكان في تطبيقنا.

```python
# ourapp/models.py

from sqlalchemy.ext.hybrid import hybrid_property

from . import bcrypt, db

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True, autoincrement=True)
    username = db.Column(db.String(64), unique=True)
    _password = db.Column(db.String(128))

    @hybrid_property
    def password(self):
        return self._password

    @password.setter
    def _set_password(self, plaintext):
        self._password = bcrypt.generate_password_hash(plaintext)
```

استخدمنا هنا المُلحق hybird من إضافة SQLAlchemy لتعريف خاصيّة تملك عدّة دوال تُستدعى من نفس المُلحَق. يُستدعى الضابِط الذي عرفناه عندما نقوم بإسناد قيمة للخاصيّة `user.password`. يُهشِر الضابِط كلمات المرور المسنودة (للخاصيّة) ويقوم بتخزينها في العمود `password_` في جدول المُستخدِم. وبما أنّنا نستخدم الخاصيّة من المُلحق hybird فيمكننا الوصول بسهولة إلى كلمة المرور المُهشَّرة لاحقاً بواسطة `user.password`.

أصبح بإمكاننا الآن كتابة دالة العرض لنُخزِّن كلمات المرور بشكل آمن وبِكُل يُسِر.

```python
# ourapp/views.py

from . import app, db
from .forms import EmailPasswordForm
from .models import User

@app.route('/signup', methods=["GET", "POST"])
def signup():
    form = EmailPasswordForm()
    if form.validate_on_submit():
        user = User(username=form.username.data, password=form.password.data)
        db.session.add(user)
        db.session.commit()
        return redirect(url_for('index'))

    return render_template('signup.html', form=form)
```

## المُصادقة

والآن بعد أن انتهينا من عمليّة تخزين كلمات المرور، وحرصنا على القيام بذلك بشكل آمن، علينا أن نتوجَّه إلى تنفيذ جزئيّة المُصادقة في التطبيق. سيكون سيناريو تسجيل الدخول كالآتي: سنصنع استمارة لنجعل المُستخدم يرسل اسمه وكلمة مروره (أو بريده الإلكتروني وكلمة مروره)، ومن ثُمَّ سنتأكَّد من أنَّ كلمة مروره التي أدخلها صحيحة. سنقوم بعد ذلك، إن صحَّ كل ما أدخله، بتعليمه على أنَّه مُستخدِم مُسجّل (ذو جلسة) عبر وضع كعكة (ملف تعريف ارتباط) في مُتصفحه. بواسطة تلك الكعكة سنستطيع تمييزه في المرّة المُقبلة التي يدخل فيها إلى الموقع.

سنقوم بدايةً بتعريف الصنف `UsernamePassword` باستخدام عناصر مكتبة `WTForms`.

```python
# ourapp/forms.py

from flask_wtf import Form
from wtforms import StringField, PasswordField
from wtforms.validators import DataRequired


class UsernamePasswordForm(Form):
    username = StringField('Username', validators=[DataRequired()])
    password = PasswordField('Password', validators=[DataRequired()])
```

سنقوم بعدها بتعريف دالة صنفيّة جديدة في نموذج قاعدة البيانات `User` لتقوم بمقارنة نص كلمة المرور وكلمة المرور المُهشَّرة الموجودة في قاعدة البيانات.

```python
# ourapp/models.py

from . import db

class User(db.Model):

    # [...] الأعمدة والخصائص الأخرى هنا

    def is_correct_password(self, plaintext)
        return bcrypt.check_password_hash(self._password, plaintext)
```

### إضافة Flask-Login

خطوتنا التالية هي إنشاء دالة العرض التي ستعرض صفحة تسجيل الدخول وستستقبل بيانات استماراتها. ستقوم الدالة، ببساطة، بمُصادقة المُستخدِم بعد التأكّد من بياناته المُدخلة عبر إضافة Flask-Login. تُبسِط تلك الإضافة عمليّة التعامل مع المُستخدمين والمُصادقة.

هناك بعض الأشياء التي علينا إعدادها قبل استخدام إضافة Flask-Login لجعلها جاهزة للعمل.

سنقوم بتعريف الدالة `login_user` في الملف <i>init</i><i>\_\_</i><i>.py</i><i>\_\_</i>.

```python
# ourapp/__init__.py

from flask_login import LoginManager

# الأكواد المتعلقة بإنشاء وإعداد التطبيق
# [...]

from .models import User

login_manager = LoginManager()
login_manager.init_app(app)
login_manager.login_view =  "signin"

@login_manager.user_loader
def load_user(userid):
    return User.query.filter(User.id==userid).first()
```

قمنا أعلاه بإنشاء كائن مُشتَق عن `LoginManager` واستهلاله بواسطة الكائن `app`، ومن ثٌمَّ تعريف المتغيّر الصنفي `login_view` الذي يُخبِر الإضافة بالدالة التي عليه الحصول بواسطتها على مُعرِّف (`id`) المُستخدم. تلك هي الأمور الأساسيّة التي عليك تعريفها قبل البدء باستخدام الإضافة.

<blockquote>
<b>ملاحظة</b><br/>
انظر إلى المزيد من <a href='https://flask-login.readthedocs.org/en/latest/#customizing-the-login-process'>الطرق لتخصيص إضافة Flask-Login</a>.
</blockquote>

الآن سنقوم بتعريف دالة العرض `signin` (التي مرّرناها سابقاً للإضافة) التي ستتولى مهمة القيام بالمُصادقة.

```python
# ourapp/views.py

from flask import redirect, url_for

from flask_login import login_user

from . import app
from .forms import UsernamePasswordForm

@app.route('/signin', methods=["GET", "POST"])
def signin():
    form = UsernamePasswordForm()

    if form.validate_on_submit():
        user = User.query.filter_by(username=form.username.data).first_or_404()
        if user.is_correct_password(form.password.data):
            login_user(user)

            return redirect(url_for('index'))
        else:
            return redirect(url_for('signin'))
    return render_template('signin.html', form=form)
```

قمنا بمُنتهى البساطة باستيراد الدالة `login_user` من الإضافة واستخدامها للتحقق من بيانات المُستخدِم عبر استدعاء `(user)login_user`. يمكنك إضافة خاصيّة لتسجيل خروج المُستخدِم الحالي عبر استدعاء الدالة `()logout_user`.

```python
# ourapp/views.py

from flask import redirect, url_for
from flask_login import logout_user

from . import app

@app.route('/signout')
def signout():
    logout_user()

    return redirect(url_for('index'))
```

## خاصيّة "إعادة تعيين كلمة المرور"

غالباً ما ستقوم بإضافة ميزة "إعادة تعيين كلمة المرور" لموقعك. تتيح هذه الميزة لمستخدميك استرجاع حساباتهم، في حال قاموا بنسيان كلمة مرورهم مثلاً، عبر بريدهم الإلكتروني. يعج تطبيق هذه الميزة بالمخاطر والثغرات المحتلمة، وهذا لأنَّ الفكرة وما فيها تقوم على جعل شخص غير مُصادَق بالوصول إلى حساب شخصي لأحد مُستخدميك. سننفذ هذه الميزة باستخدام تكتيكات مماثلة للتي استخدمناها في قسم "تأكيد البريد الإلكتروني".

سنحتاج إلى استمارتين: واحدة لطلب استرجاع الحساب الشخصي وأخرى لاختيار كلمة المرور الجديدة بعد التأكّد من أنَّ صاحب الطلب لديه وصول لبريد الحساب الإلكتروني. أفترض في هذا القسم أنَّ نموذج قاعدة بياناتك يحوي عموداً لكلمة المرور وآخر للبريد الإلكتروني، حيث أنَّ عمود كلمات المرور يستخدم خاصيّة hybird كالتي أنشأناها سابقاً.

<blockquote>
<b>ملاحظة</b><br/>
لا تُرسِل رابط لإعادة تعيين كلمة مرور إلى بريد إلكتروني غير مؤكَّد، فعليك الحرص أنَّك تُرسِل الرابط للشخص المُناسِب.
</blockquote>

يوجد ادناه تعريف للصنفين الذين سنستخدمهما في الاستماراتين اللتين سنحتاجهما لتنفيذ الخاصيّة.

```python
# ourapp/forms.py

from flask_wtf import Form
from wtforms import StringField, PasswordField
from wtforms.validators import DataRequired, Email

class EmailForm(Form):
    email = StringField('Email', validators=[DataRequired(), Email()])

class PasswordForm(Form):
    password = PasswordField('Password', validators=[DataRequired()])
```

تفترض الشيفرة أعلاه أنّنا سنستخدم حقلاً واحداً في استمارة إدخال كلمة المرور الجديدة. تَطلُب العديد من التطبيقات من مُستخدميها إدخال كلمات مرورهم الجديدة مرّتين للتتأكّد أنَّهم لم يقعوا في أيّة أخطاء إملائيّة أثناء كتابتها. تحقيق ذلك لن يُكلِّف الكثير، فقط أضف حقلاً آخراً من النوع `PasswordField` وأضف المُصادِق `EqualTo` إلى حقل إدخال كلمة المرور الأوّل.

<blockquote>
<b>ملاحظة</b><br/>
توجد العديد من المُناقشات المفيدة في مجتمع "تجربة المُستخدِم" حول أفضل طريقة يُمكِن أن تُستخدم للتعامل مع استمارات تسجيل الدخول. أُفضِل شخصياً أفكار المُستخدِم روجر أتريل (Roger Attrill) الذي يقول:
<br/><br/>
"لا يجدر بنا السؤال عن كلمة المرور مرّتين، فينبغي علينا أن نسأل عنها مرَّة واحدة ونتأكّد أن تعمل ميزة 'إعادة تعيين كلمة المرور' بسلاسة ومثاليّة."
<br/><br/>
<ul style='list-style-type: disc; list-style-position: inside;'>
<li>اقرأ المزيد حول هذا الشأن في <a href='http://ux.stackexchange.com/questions/20953/why-should-we-ask-the-password-twice-during-registration/21141'>موضوعه في مجتمع تجربة المُستخدِم</a>.</li>
<li>يوجد أيضاً بعض الأفكار الرائعة حول تبسيط استمارات تسجيل الدخول وإنشاء الحساب <a href='http://uxdesign.smashingmagazine.com/2011/05/05/innovative-techniques-to-simplify-signups-and-logins/'>في هذا الموضوع</a>.</li>
</ul>
</blockquote>

سنقوم الآن بكتابة دالة العرض للاستمارة الأولى، والتي يقوم فيها المُستخدِم بطلب إرسال رابط لاسترجاع حسابه إلى بريده الإلكتروني.

```python
# ourapp/views.py

from flask import redirect, url_for, render_template

from . import app
from .forms import EmailForm
from .models import User
from .util import send_email, ts

@app.route('/reset', methods=["GET", "POST"])
def reset():
    form = EmailForm()
    if form.validate_on_submit():
        user = User.query.filter_by(email=form.email.data).first_or_404()

        subject = "Password reset requested"

        # هنا حيث سنقوم باستخدام الصنف URLSafeTimedSerializer الذي قُمنا بإنشائه
        # في بداية الفصل.
        token = ts.dumps(user.email, salt='recover-key')

        recover_url = url_for(
            'reset_with_token',
            token=token,
            _external=True)

        html = render_template(
            'email/recover.html',
            recover_url=recover_url)

        # دعنا نفترض أنَّ هذه الدالة (send_eamil) تم تعريفها في الوحدة myapp/util.py
        send_email(user.email, subject, html)

        return redirect(url_for('index'))
    return render_template('reset.html', form=form)
```

ستقوم الاستمارة عند تلقيها للبريد الإلكتروني بتحديد المُستخدم (عبر البريد المُدخَل)، ومن ثُمَّ ستولِّد رمزاً فريداً وستُرسِل للبريد الإلكتروني رابطاً لإعادة تعيين كلمة المرور. سيوجّه الرابط صاحب البريد إلى دالة عرض لتقوم بالتأكّد من صلاحيّة الرمز وتجعل المُستخدِم من بعدها يُعيِّن كلمة مروره الجديدة.

```python
# ourapp/views.py

from flask import redirect, url_for, render_template

from . import app, db
from .forms import PasswordForm
from .models import User
from .util import ts

@app.route('/reset/<token>', methods=["GET", "POST"])
def reset_with_token(token):
    try:
        email = ts.loads(token, salt="recover-key", max_age=86400)
    except:
        abort(404)

    form = PasswordForm()

    if form.validate_on_submit():
        user = User.query.filter_by(email=email).first_or_404()

        user.password = form.password.data

        db.session.add(user)
        db.session.commit()

        return redirect(url_for('signin'))

    return render_template('reset_with_token.html', form=form, token=token)
```

سنستخدم نفس طريقة التأكَّد من الرمز التي استخدمناها سابقاً مع ميزة تأكيد البريد الإلكتروني. حيث ستقوم دالة العرض بتمرير رمز التأكيد من الرابط إلى القالب، وبعدها سيستخدم القالب الرمز لإرسال الاستمارة إلى دالة العرض. إليك الكود المُستخدَم في القالب:

```html
{# ourapp/templates/reset_with_token.html #}

{% extends "layout.html" %}

{% block body %}
<form action="{{ url_for('reset_with_token', token=token) }}" method="POST">
    {{ form.password.label }}: {{ form.password }}<br>
    {{ form.csrf_token }}
    <input type="submit" value="Change my password" />
</form>
{% endblock %}
```

## الخلاصة

<ul style='list-style-type: disc; list-style-position: inside;'>
<li>استخدم مكتبة <code>itsdangerous</code> لإنشاء والتأكّد من صلاحيّة رموز التأكيد المُرسلة إلى بريد الإلكتروني لمُستخدميك.</li>
<li>يُمكِنك استخدام طريقة رموز التأكيد للتحقق من البريد الإلكتروني عندما يقوم المُستخدِم بإنشاء حسابه أو تعديل بريده أو استرجاع كلمة مروره.</li>
<li>استخدم إضافة Flask-Login لمُصادِقة المُستخدمين لكي تتجنب إدارة بيانات الجلسة يدوياً.</li>
<li>فكِّر دائماً بعقليّة المهاجم الذي قد يستغل تطبيقك لإحداث الأضرار.</li>
</ul>
