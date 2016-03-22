# Flask SQLite

Last week we looked at how to combine HTM/CSS with the Python language using Jinja2 templating system. This combination is very powerful as it overcomes some limitations associated with HTML such as the lack of flow control mechanims. 

This week we'll rely Python language's SQL handling capabilites to build websites that is capable of connecting/manipulating daat in a relational database. The excercises that follow conclude the current module.

## Forms in Flask

Before you manipulate data using SQLite, you need to collect user inputs using forms. There're several different ways of presenting forms in Flask, obviously they all have pros/cons. Let's examine them all.

### Pure HTML forms

In your C9 workspace, create a new file called 'form_simple.html' and put it in a folder called 'tempaltes' (note: ploral, not template). Insert the following code into this file.

```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>A very simple form</title>
</head>
<body>
<form action="" method="post">
    <label for="nameId">Name:</label>
    <br>
    <input type="text" name="name" id="nameId">
    <br>
    <label for="commentsId">Comments:</label>
    <br>
    <textarea name="comments" id="commentsId" cols="30" rows="10"></textarea>
    <br>
    <input type="submit" value="Submit">
</form>

{% if name %}
    <p> Your name is: {{ name }}</p>
    <p>and your comments are: {{ comments }}</p>
    <p>A big thank you!</p>
{% endif %}

</body>
</html>
```

If you did last week's excercises, this is easy to understand -- it's just a simple Jinja2 template with a standart HTML form in it.

Next, in workspace folder create a file called 'form_simple.py' and insert the following code

```python
from flask import Flask, render_template, request

app = Flask(__name__)


@app.route('/', methods=['GET', 'post'])
def view_form():
    if request.method == 'POST':
        name = request.form['name'];
        comments = request.form['comments']
        return render_template('form_simple.html', name=name, comments=comments)
    else:
        return render_template('form_simple.html')


if __name__ == '__main__':
    app.run(port=8080, host='0.0.0.0', debug=True)
```

The majority of the code should look familiar, apart from the following two:

1. The function decorator takes an additional 'method' argument. By defaul (without the 'method' argument), the current route i.e. `/` will only respond to `GET` method. That's what happens when people try to view a webpages. However, when they try to send data in a form, they will have to specify either 'POST' or 'PUT' in the form's `method` attribute. When you design your route, you'll need to respond to that **HTTP method**.
2. 'request' here is an object that encapsulates the contents of a HTTP request sent by the client (normally a browser). Here we use it to access the data contained in the form. 

If you run the code, type in something and hit 'submit', what you'll see is similar to below. The info you just typed in has be displayed back.

![](.md_images/simple.png)

> Last week we had some problems with C9, and I suggested that you register a free tiral account on Codio that lasts for 30 days (only). In case needed, you may consider get [PyCharm](https://www.jetbrains.com/student/) installed on your desktop, which normally costs £149 a year but is free for educational purposes. At the moment you don't have to do this as C9 came back to normal.

> ![](.md_images/charm.png)

### Bootstrap forms

Obviously pure HTML forms work, but as you can see it doesn't look very nice. Using the Flask-Bootstrap extension we can take advantage of existing CSS rules in Bootstrap.


Create a new file called 'form_bootstrap.py', and insert the following code

```python
from flask import Flask, render_template, request
from flask_bootstrap import Bootstrap

app = Flask(__name__)
Bootstrap(app)

@app.route('/')
def view_form():
    return render_template('form_bootstrap.html')


if __name__ == '__main__':
    app.run(port=8080, host='0.0.0.0', debug=True)
```

This is a standard Flask app using Bootstrap, the same as what we saw in the previos lab. 

Next, create a file called 'form_bootstrap.html' and put into the templates folder. Insert the following lines into this newly created file.

```html
{% extends 'bootstrap/base.html' %}

{% block title %}
    A simple form using Bootstrap
{% endblock %}

{% block content %}
    <div class="container">
        <form action="" method="post">
            <div class="form-group">
                <label for="nameId">Name:</label>
                <input type="text" name="name" id="nameId" class="form-control" placeholder="Name">
            </div>
            <div class="form-group">
                <label for="commentsId">Comments:</label>
                <textarea name="comments" id="commentsId" rows="8" class="form-control"></textarea>
            </div>

            <button type="submit" class="btn btn-default">Submit</button>
        </form>
    </div>

{% endblock %}
```

This HTML template has the same number of input fields as the pure HTML example, but it uses the Bootstrap format:

* The form is being contained in a 'container' class.
* Input fields and associated labels are groupled using the 'form-group' class.
* Input fileds are marked using the 'form-control' class.
* Submit button has been assigned the 'btn' and 'btn-default' classes.

If you run the code you'll see that the form looks much better as it inherits all Bootstrap rules and interactivities.

![](.md_images/xxx.png)

> The recommendation from Bootstrap is to *"void using <textarea> elements as their rows attribute will not be respected in some cases"*, see [here](http://getbootstrap.com/components/#input-groups).


### Forms using WTF

Bootstrap was covered earlier in the module, in lab12. The only reason we need to re-visit is to use Flask extension Flask-WTF, which offers a simple wrapper around WTForms. WTForms is a flexible forms validation and rendering library for python web development that turns the whole form into classes and objects.

Create a new file and name it 'form_wtf.py'. Insert the following code into it

```python
from flask import Flask, render_template
from flask_bootstrap import Bootstrap
from flask_wtf import Form
from wtforms import StringField, TextAreaField, SubmitField
from wtforms.validators import DataRequired, Length

app = Flask(__name__)
app.config['SECRET_KEY'] = 'hard to guess string'
Bootstrap(app)


class CommentForm(Form):
    name = StringField('Name:', validators=[DataRequired()])
    comments = TextAreaField('Comments', validators=[DataRequired(), Length(min=3, max=10)])
    submit = SubmitField('Submit')


@app.route('/', methods=['GET', 'POST'])
def view_form():
    form = CommentForm()
    if form.validate_on_submit():
        name = form.name.data
        comments = form.comments.data
        form.name.data = ''
        form.comments.data = ''
        return render_template('form_wtf.html', form=form, name=name, comments=comments)
    return render_template('form_wtf.html', form=form)


if __name__ == '__main__':
    app.run(port=8080, host='0.0.0.0', debug=True)
```

There are quite a lot of things going on in the example above:

* We created a class called CommentForm that inherits from the Form class that comes with package flask_wtf. This class is being used as the form in an ordinary HTML context.
* This CommentForm class has three attributes, corresponding to the three form controls we have. Note here these attributes are objects themselve, imported from a different module called 'wtforms'.
* Form controls i.e. fields can have validators. These are imported classes. The differences between this Python validators and JS validators we saw earlier in the module is that Python validators need to be run on the server. That is, the form is sent first to get validation.
* Form objects have an instance funciton called `validate_on_submit()`. Basically if the form validates fine, this function will evaluate to true, and the sub-block will get run
> If you Google online, you'll notice that some tutorial use 'data_required()' or 'required()' instead of 'DataRequired()'. In fact, 'data_required' is the same as 'DataRequired()', and  'required()' is the legacy version of the same thing.

Now, create a file called 'form_wtf.html' and put inside the templates folder. Insert the following lines into this file

```html
{% extends 'bootstrap/base.html' %}
{% import 'bootstrap/wtf.html' as wtf %}

{% block title %}
    A simple form using WTF
{% endblock %}

{% block content %}
    <div class="container">
        {{ wtf.quick_form(form) }}

        {% if name %}
            <p> Your name is: {{ name }}</p>
            <p>and your comments are: {{ comments }}</p>
            <p>A big thank you!</p>
        {% endif %}
    </div>

{% endblock %}
```

## SQLite integration

### Collect user inputs

### Save data in SQLite

## Advanced


> ![](.md_images/flash.jpg)

> There's a new book came out recently using [Zootopia's Flash](http://disney.wikia.com/wiki/Flash) as cover image. Grab a copy of the book and see if it makes differences.


### Flask-SQLAlchemy

### Flask-Mail
