# الملفات الساكنة

<img src='../images/static.png'/>

كما يوضِّح اسمها، الملفات الساكنة هي الملفات التي لا تتغيّر. في تطبيقك العادي، ستجد ملفات ساكنة مثل ملفات CSS وملفات الجافاسكربت والصور. قد يتضمن تطبيقك أيضاً ملفات الصوت وغيرها من الأشياء المشابهة.

## تنظيم الملفات الساكنة

سننشئ دليلاً لملفاتنا الساكنة وسنسميه *static* بداخل حزمة تطبيقنا.

```
myapp/
    __init__.py
    static/
    templates/
    views/
    models.py
run.py
```

كيفيّة تنظيم الملفات بداخل الدليل `/static` هي تفضيل شخصي بحت. شخصياً، أنزعج قليلاً من انمزاج مكتبات الطرف الثالث (مثل جيكوري وبوتستراب) مع ملفات CSS والجافاسكربت خاصتي. لتجنب ذلك أنصح بفصل مكتبات الطرف الثالث ووضعها بداخل المجلد */lib* في دليل مناسب لها. بعض المشاريع تستخدم الدليل */vendor* بدلاً من */lib*.

```
static/
    css/
        lib/
            bootstrap.css
        style.css
        home.css
        admin.css
    js/
        lib/
            jquery.js
        home.js
        admin.js
    img/
        logo.svg
        favicon.ico
```

### عرض أيقونة المفضلة

ستُعرَض الملفات الساكنة بداخل الدليل *static* من الرابط */example.com/static*. تتوقع متصفحات الويب وغيرها من البرمجيات وجود أيقونة المفضلة (favicon) في الرابط *example.com/favicon.ico* افتراضياً. لإصلاح هذا التضارب، يمكننا إضافة السطر التالي إلى قسم `<head>` في قالب موقعنا.

```html
<link rel="shortcut icon"
    href="{{ url_for('static', filename='img/favicon.ico') }}">
```

## إدارة الأصول (الملفات) الساكنة باستخدام إضافة Flask-Assets

إضافة Flask-Assets هي إضافة متخصصة بإدارة الملفات الساكنة. تتيح هذه الإضافة أداتان مفيدتان بشكل كبير. تسمح لك الأداة الأولى بتعريف **رزم** (bundles) من الأصول في شيفرة بايثون خاصتك يُمكِن إدراجها معاً في قالب موقعك. أما الأداة الثانية فتسمح لك **بالمعالجة المسبقة** (pre-process) لتلك الملفات. هذا يعني أنه يمكنك مزج وتصغير ملفات الـ CSS والجافاسكربت خاصتك بحيث لن يكون على المستخدم سوى تحميل ملفين مُصغرين من دون إجبارك على تطوير خط ضخ معقد. يمكنك حتى تجميع (compile) ملفاتك من مصادر Sass، و LESS، و CoffeeScript وغيرها من المصادر الأخرى.

```
static/
    css/
        lib/
            reset.css
        common.css
        home.css
        admin.css
    js/
        lib/
            jquery-1.10.2.js
            Chart.js
        home.js
        admin.js
    img/
        logo.svg
        favicon.ico
```

### تعريف الرزم

يملك تطبيقنا قسمين: الموقع العمومي ولوحة تحكم المشرف، ويُشار إليهم بـ "home" و "admin" على التوالي في تطبيقنا. سنُعرِف أربع رزم تغطي ملفات CSS والجافاسكربت لكل قسم. سنُعرِف الرزم في الوحدة *assets* بداخل الحزمة `util`.

```python
# myapp/util/assets.py

from flask_assets import Bundle, Environment
from .. import app

bundles = {

    'home_js': Bundle(
        'js/lib/jquery-1.10.2.js',
        'js/home.js',
        output='gen/home.js'),

    'home_css': Bundle(
        'css/lib/reset.css',
        'css/common.css',
        'css/home.css',
        output='gen/home.css'),

    'admin_js': Bundle(
        'js/lib/jquery-1.10.2.js',
        'js/lib/Chart.js',
        'js/admin.js',
        output='gen/admin.js'),

    'admin_css': Bundle(
        'css/lib/reset.css',
        'css/common.css',
        'css/admin.css',
        output='gen/admin.css')
}

assets = Environment(app)

assets.register(bundles)
```

تقوم الإضافة بمزج ملفاتك حسب الترتيب الذي تم سردهم به. فإذا كان الملف *admin.js* يتطلب الملف *jquery-1.10.2.js* احرص على وضع ملف الجيكوري أولاً.

قمنا بتعريف الرزم في دليل لجعل تسجليهم أسهل. الحزمة webassets، والتي تعتمد عليها إضافة Flask-Assets تتيح لنا تسجيل الرزم بعدة طرق، ومنها تمرير قاموس كالذي استخدمناه في المقتطف السابق.<sup>[1]</sup>

في حين أننا نُسجِل رزمنا في `util.assets`، فكل ما علينا القيام به هو استيراد تلك الوحدة في الملف <i>init</i><i>\_\_</i><i>.py</i><i>\_\_</i> بعد استهلال تطبيقنا.

```python
# myapp/__init__.py

# [...] هنا نقوم باستهلال التطبيق

from .util import assets
```

### استخدام الرزم

لاستخدام الرزم الخاصة بقسم لوحة المشرف، سنقوم بإدراجهم في القالب الأب لقسم المشرف: *admin/layout.html*.

```
templates/
    home/
        layout.html
        index.html
        about.html
    admin/
        layout.html
        dash.html
        stats.html
```

```html
{# myapp/templates/admin/layout.html #}

<!DOCTYPE html>
<html lang="en">
    <head>
        {% assets "admin_js" %}
            <script type="text/javascript" src="{{ ASSET_URL }}"></script>
        {% endassets %}
        {% assets "admin_css" %}
            <link rel="stylesheet" href="{{ ASSET_URL }}" />
        {% endassets %}
    </head>
    <body>
    {% block body %}
    {% endblock %}
    </body>
</html>
```

يمكننا القيام بنفس الشيء لرزم القسم "home" في الملف *templates/home/layout.html*.

### استخدام المُرشِّحات

يمكننا استخدام المُرشِّحات للقيام بمعالجة مسبقة لملفاتنا الساكنة. هذا مفيد بشكل خاص لتصغير رزم CSS والجافاسكربت خاصتنا.

```python
# myapp/util/assets.py

# [...]

bundles = {

    'home_js': Bundle(
        'lib/jquery-1.10.2.js',
        'js/home.js',
        output='gen/home.js',
        filters='jsmin'),

    'home_css': Bundle(
        'lib/reset.css',
        'css/common.css',
        'css/home.css',
        output='gen/home.css',
        filters='cssmin'),

    'admin_js': Bundle(
        'lib/jquery-1.10.2.js',
        'lib/Chart.js',
        'js/admin.js',
        output='gen/admin.js',
        filters='jsmin'),

    'admin_css': Bundle(
        'lib/reset.css',
        'css/common.css',
        'css/admin.css',
        output='gen/admin.css',
        filters='cssmin')
}

# [...]
```

<blockquote>
<b>ملاحظة</b><br/>
لاستخدام المُرشِّحات <code>jsmin</code> و <code>cssmin</code>، احرص على تثبيت الحزمتين <code>jsmin</code> و <code>cssmin</code> (على سبيل المثال باستخدام <code>pip install jsmin cssmin</code>). تأكد من إضافتهم إلى ملف المتطلبات (<i>requirements.txt</i>) أيضاً.
</blockquote>

ستقوم إضافة Flask-Assetes بدمج وضغط ملفاتنا في أول مرة يُعرَض فيها القالب، وستقوم تلقائياً بتحديث الملفات المضغوطة عندما يتم تحديث أحد ملفات المصدر.

<blockquote>
<b>ملاحظة</b><br/>
إذا قمت بإعطاء الخيار <i>ASSETS_DEBUG</i> القيمة <i>True</i> في ملف الإعداد خاصتك فستقوم الإضافة بإخراج كل ملف مصدر مفرداً بدلاً من دمجهم.
</blockquote>

<blockquote>
<b>ملاحظة</b><br/>
ألقي نظرة على بعض <a href='http://elsdoerfer.name/docs/webassets/builtin_filters.html#js-css-compilers'>المُرشِّحات الأخرى</a> التي يمكنك استخدامها مع إضافة Flask-Assets.
</blockquote>

## الخلاصة

<ul style='list-style-type: disc; list-style-position: inside;'>
  <li>توضع الملفات الساكنة في الدليل <i>/static</i>.</li>
  <li>قم بفصل مكتبات الطرف الثالث عن الملفات الساكنة خاصتك.</li>
  <li>حدد موقع أيقونة المفضلة في قوالبك.</li>
  <li>استخدم إضافة Flask-Assets لإدراج الملفات الساكنة في قوالبك.</li>
  <li>تتمكن إضافة Flask-Assets من تجميع، ودمج، وضغط ملفاتك الساكنة.</li>
</ul>

## المصادر والمراجع

<ol style='list-style-type: decimal; list-style-position: inside;'>
  <li>يمكنك رؤية كيف تعمل عملية تسجيل الرزم <a href='https://github.com/miracle2k/webassets/blob/0.8/src/webassets/env.py#L380'>في الشيفرة المصدريّة للإضافة</a>.</li>
</ol>
