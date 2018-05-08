# القوالب

<img src='../images/templates.png'/>

في حين أنَّ فلاسك لا يجبرك على استخدام أيَّة لغة قوالب محددة، فهو يفترض أنك ستستخدم لغة جينجا (Jinja). معظم المطورين في مجتمع فلاسك يستخدمون جينجا، وأنصحك بأن تفعل المِثل. يوجد بضعة إضافات (ملحقات) تتيح لنا استخدام لغات قوالب أخرى، مثل [Flask-Genshi](http://pythonhosted.org/Flask-Genshi/) و [Flask-Mako](http://pythonhosted.org/Flask-Mako/). استخدم الخيار الإفتراضي (جينجا) ما لم يكن لديك سبب جيد لتستخدم شيء آخر. عدم معرفتك للبنية الكتابية لجينجا ليس بالسبب الجيد! فالسوف توفر الكثير من الوقت والصداع.

<blockquote>
<b>ملاحظة</b><br/>
معظم المواقع تَقصُد Jinja2 في حين أنها تكتب "Jinja" وحسب. حيث كان هناك إصدار أول من هذه اللغة (Jinja1)، ولكننا لن تعامل معه هنا. عندما سأذكر جينجا في هذا الكتب، فسأكون أقصد <a href='http://jinja.pocoo.org'>هذا الإصدار</a>.
</blockquote>

## مقدمة سريعة حول جينجا

يؤدي توثيق جينجا عملاً جيداً في شرح البنية الكتابية وميزات هذه اللغة. لن أقوم بإعادة المعلومات المذكورة هناك، ولكن أريد أن أحرص على رؤيتك لهذه الملاحظة الهامة:

<blockquote>
 يوجد نوعان من المحددات في اللغة: <code>{% raw %}{% ... %}{% endraw %}</code> و <code>{% raw %}{{ ... }}{% endraw %}</code>. يُستخدم الأول لتنفيذ الجمل (Statements) مثل حلقة for أو لإسناد القيم، أما الثاني فيُستخدم لطباعة نتيجة تعبير إلى القالب.

<br/><br/>
— <a href='http://jinja.pocoo.org/docs/templates/#synopsis'>توثيق مُصمم قوالب جينجا</a>
</blockquote>

## كيفية تنظيم القوالب

أين المكان المناسب لوضع القوالب في تطبيقنا؟ إذا كنت قد تابعت الدورة منذ البداية، فمن المؤكد أنك لاحظتَ أنَّ إطار فلاسك يملك مرونة عالية فيما يتعلق بتنظيم الأشياء. ولربما لاحظت أيضاً أنَّ هناك عادةً مكان موصى به لوضع تلك الأشياء. يوصى بوضع القوالب في دليل الحزمة.

```myapp/
    __init__.py
    models.py
    views/
    templates/
    static/
run.py
requirements.txt

```

```
templates/
    layout.html
    index.html
    about.html
    profile/
        layout.html
        index.html
    photos.html
    admin/
        layout.html
        index.html
        analytics.html
```

يماثل تنظيم الدليل *templates* تنظيم موجهاتنا. حيث أنَّ القالب الذي سيُعرض في العنوان *myapp.com/admin/analytics* هو *templates/admin/analytics.html*. كما يوجد بعض القوالب الإضافية التي ستُستَمَد ولكن لن يتم عرضها بشكل مباشر. بحيث أنَّ ملفات *layout.html* سيتم وراثتها (استمدادها/تضمينها) من قوالب أخرى.

## الوراثة

يعتمد دليل القوالب الُمنظم بشكل جيد بشدة على الوراثة. فيُعرِف **القالب الأب** عادةً الهيكل العام الذي سترثه جميع **القوالب الأبناء**. في مثالنا السابق، يعد القالب *layout.html* هو القالب الأب، والملفات البقيّة التي تحمل الإمتداد *html.* هي القوالب الأبناء.

ستملك عادةً ملف *layout.html* رئيسي (ذا مستوى أول) يُعرِف التخطيط (التصميم/التنظيم) العام لتطبيقك، وملف لكل قسم من موقعك. إذا قمت بأخذ نظرة على الدليل أعلاه، ستجد أنَّ هناك قالب رئيسي (*myapp/templates/layout.html*) كما توجد قوالب رئيسية فرعية مثل *myapp/templates/profile/layout.html* و *myapp/templates/admin/layout.html*. القالبان الفرعيان يرثان ويعدلان الأول (القالب ذو المستوى الأول).

يتم تنفيذ الوراثة باستخدام الوسمان `{% extends %}` و `{% block %}`. حيث يمكننا تعريف كتل في القالب الأب ليتم ملؤها (الكتابة بداخلها) بواسطة القوالب الأبناء.

```html
{# _myapp/templates/layout.html_ #}

<!DOCTYPE html>
<html lang="en">
    <head>
        <title>{% block title %}{% endblock %}</title>
    </head>
    <body>
    {% block body %}
        <h1>This heading is defined in the parent.</h1>
    {% endblock %}
    </body>
</html>
```

الآن يمكننا استمداد (Extend) القالب الأب في القالب الابن وتعريف محتوى (ملء) هذه الكتل.

```html
{# _myapp/templates/index.html_ #}

{% extends "layout.html" %}
{% block title %}Hello world!{% endblock %}
{% block body %}
    {{ super() }}
    <h2>This heading is defined in the child.</h2>
{% endblock %}
```

تسمح لنا الدالة `()super` بتضمين أياً كان ما بداخل هذه الكتلة في القالب الأب.

<blockquote>
<b></b><br/>
للمزيد من المعلومات، قم بزيارة <a href='http://jinja.pocoo.org/docs/templates/#template-inheritance'>وثيقة جينجا عن الوراثة في القوالب</a>.
</blockquote>

## إنشاء وحدات ماكرو

يمكننا تطبيق مبدأ [لا تكرر نفسك](https://ar.wikipedia.org/wiki/%D9%84%D8%A7_%D8%AA%D9%83%D8%B1%D8%B1_%D9%86%D9%81%D8%B3%D9%83) في القوالب عن طريق استخلاص المقاطع البرمجية التي ستُستخدم بشكل متكرر وتحويلها إلى **وحدات ماكرو** (Macros). على سبيل المثال، إن كنا نعمل على الشريط العلوي لتطبيقنا، فقد نرغب بإعطاء صنف (Class) مختلف للرابط "النشط" (أي رابط الصفحة الحالية). من دون استخدام وحدات الماكرو، سنضطر إلى استخدام العبارة `if ... else` للتحقق من كل رابط لإيجاد النشط منهم.

توفر وحدات الماكرو طريقة لتجزئة الشيفرة البرمجية، فهي تعمل كالدوال. دعنا نلقي نظرة على طريقة يمكننا استخدامها لتحديد الرابط النشط باستخدام الماكرو.

```html
{# myapp/templates/layout.html #}

{% from "macros.html" import nav_link with context %}
<!DOCTYPE html>
<html lang="en">
    <head>
    {% block head %}
        <title>My application</title>
    {% endblock %}
    </head>
    <body>
        <ul class="nav-list">
            {{ nav_link('home', 'Home') }}
            {{ nav_link('about', 'About') }}
            {{ nav_link('contact', 'Get in touch') }}
        </ul>
    {% block body %}
    {% endblock %}
    </body>
</html>
```

ما قُمنا به في هذا المثال هو استدعاء ماكرو غير مُعرَّف (وهو الماكرو `nav_link`) وتمرير معاملين له: الأول اسم الدالة الهدف (أي اسم الدالة لدالة العرض المستهدفة) والنص الذي نريد إظهاره.

<blockquote>
<b>ملاحظة</b><br/>
قد تكون لاحظت أننا استخدامنا <code>with context</code> في عبارة الاستدعاء. يتكون <b>سياق</b> (Context) جينجا من مجموعة المعاملات المُمررة إلى دالة <code>()rendrer_template</code> وإلى سياق بيئة جينجا (Jinja environment context) من الشيفرة البرمجية. تُصبح هذه المتغيرات متوفرة في القالب الذي يجري عرضه.
<br/><br/>
بعض المتغيرات تُمرر بشكل جلي بواسطتنا، مثل <code>render_template("index.html", color="red")</code>، ولكن يوجد بعض المتغيرات والدوال تُضمَن تلقائياً بواسطة إطار فلاسك، مثل <code>request</code> و <code>g</code> و <code>session</code>. فعندما نستخدم <code>{% raw %}{% from ... import ... with context %}{% endraw %}</code> نحن نخبر مُحرِك جينجا أن يجعل جميع هذه المتغيرات متوفر أيضاً في الماكرو.
</blockquote>

<blockquote>
<b>ملاحظة</b><br/>
<ul style='list-style-type: disc; list-style-position: inside;'>
<li>يمكنك إيجاد جميع المتغيرات العمومية (global) المُمرّرة إلى سياق جينجا تلقائياً من <a href='http://flask.pocoo.org/docs/templating/#standard-context'>هنا</a>.</li>
<li>يمكننا تعريف دوال ومتغيرات نريد دمجها في سياق جينجا باستخدام <a href='http://flask.pocoo.org/docs/templating/#context-processors'>معالجات السياق</a>.</li>
</ul>
</blockquote>

الآن حان وقت تعريف الماكرو `nav_link` الذي استخدمناه في ذلك القالب.

```html
{# myapp/templates/macros.html #}

{% macro nav_link(endpoint, text) %}
{% if request.endpoint.endswith(endpoint) %}
    <li class="active"><a href="{{ url_for(endpoint) }}">{{text}}</a></li>
{% else %}
    <li><a href="{{ url_for(endpoint) }}">{{text}}</a></li>
{% endif %}
{% endmacro %}
```

الآن قمنا بتعريف الماكرو في المسار *myapp/templates/macros.html*. في هذا الماكرو قمنا باستخدام الكائن `request` - والذي يكون متوفر إفتراضياً في سياق جينجا - للتأكد مما إذا كان الطلب الحالي قد وُجِّهَ إلى الدالة المُمرّرة إلى الماكرو `nav_link`. إن كان، فبالتالي نحن في تلك الصفحة ويمكننا تعليم رابطها كرابط نشط.

<blockquote>
<b>ملاحظة</b><br/>
عبارة <code>from x import y</code> تأخذ قيمة x كمسار نسبي. بحيث إذا كان قالبنا في المسار <i>myapp/templates/user/blog.html</i> فسنستخدم <code>from "../macros.html" import nav_link with context</code>.
</blockquote>

## مرشحات مخصصة

مرشحات جينجا هي عبارة عن دوال يُمكِن تطبيقها على نتائج العبارة في المحدد <code>{% raw %}{{ ... }}{% endraw %}</code>. بحيث تُطبَق هذه المُرشحات قبل طباعة هذه النتائج في القالب.

```html
<h2>{{ article.title|title }}</h2>
```

في المثال أعلاه، المرشح `title` سيأخذ `article.title` ويعيد النسخة المعنونة حرفياً (title cased) منه، والتي من ثم ستُطبَع إلى القالب. تشبه آلية عمل هذه الخاصية إلى حدٍ كبير آلية عمل "ضخ (piping)" مُخرج برنامج إلى مدخلات برنامج آخر في أنظمة يونكس.

<blockquote>
<b>ملاحظة</b><br/>
توجد حفنة من المُرشحات المبنية ضمنياً في جينجا من مثل <code>title</code>. يمكنك رؤية <a href='http://jinja.pocoo.org/docs/templates/#builtin-filters'>القائمة الكاملة</a> في توثيقات جينجا.
</blockquote>

يمكننا تعريف مُرشِحات خاصة بنا لاستخدامها في قوالب جينجا. كمثال، سنقوم بتنفيذ مرشح بسيط باسم `caps` يقوم بجعل جميع أحرف السلسلة النصية كبيرة.

<blockquote>
<b>ملاحظة</b><br/>
يملك محرك جينجا بالفعل مرشح يسمى <code>upper</code> وهو يؤدي تلك الوظيفة، كما يملك المرشح <code>capitalize</code> الذي يقوم بجعل الحرف الأول بالحالة الكبيرة وباقي الأحرف بالحالة الصغيرة. تتعامل هذه المرشحات أيضاً مع تحويلات اليونيكود (Unicode)، ولكننا سنبقي المثال بسيطاً لتسليط الضوء على الفكرة.
</blockquote>

سنقوم بتعريف المرشح في وحدة تتوضع في المسار *myapp/util/filters.py*. أي سيصبح لدينا الحزمة `util` والتي سنضع فيها الوحدات الأخرى المتنوعة.

```python
# myapp/util/filters.py

from .. import app

@app.template_filter()
def caps(text):
    """Convert a string to all caps."""
    return text.uppercase()
```

في هذا المثال قمنا بتسجيل الدالة كمرشح جينجا باستخدام المُزخرِف `()app.template_filter@`. يأخذ المرشح اسمه الإفتراضي من اسم الدالة، ولكن يمكننا تغيير ذلك عن طريق تمرير معامل للمُزخرِف.

```python
@app.template_filter('make_caps')
def caps(text):
    """Convert a string to all caps."""
    return text.uppercase()
```

الآن يمكننا استدعاء المرشح بالاسم `make_caps` في القالب بدلاً من استخدام `caps` كالتالي: `{{ "hello world!"|make_caps }}`.

لجعل المشرح متاحاً في القوالب، كل ما علينا القيام به هو استدعاءه في ملف <i>init</i><i>\_\_</i><i>.py</i><i>\_\_</i> الرئيسي.

```python
# myapp/__init__.py

# تأكد أنه تم استهلال التطبيق أولاً لمنع مشاكل الاستيرادات الحلقية.
from .util import filters
```

## الخلاصة

<ul style='list-style-type: disc; list-style-position: inside;'>
<li>استخدم جينجا للتعامل مع القوالب.</li>
<li>يوجد نوعان من المحددات في محرك جينجا: <code>{% raw %}{% ... %}{% endraw %}</code> و <code>{% raw %}{{ ... }}{% endraw %}</code>. يُستخدم الأول لتنفيذ الجمل (Statements) مثل حلقة for أو لإسناد القيم، أما الثاني فيُستخدم لطباعة نتيجة تعبير إلى القالب.</li>
<li>ينصح بأن توضع القوالب في الدليل <i>myapp/templates/</i> أي في دليل بداخل حزمة التطبيق.</li>
<li>أنصح بأن تعكس هيكلة الدليل <i>templates/</i> هيكلة عناوين الروابط في التطبيق.</li>
<li>أنصح بأن تملك ملف <i>layout.html</i> رئيسي في الدليل <i>myapp/templates</i> وملف خاص لكل قسم في الموقع. وعليك أن تحرص على أن يستدعي الملف المخصص لقسم معين الملف الرئيسي.</li>
<li>وحدات الماكرو هي مثل الدوال مصنوعة من شيفرة القالب.</li>
<li>المرشحات هي دوال مصنوعة من شيفرة بايثون وتستخدم في القوالب.</li>
</ul>
