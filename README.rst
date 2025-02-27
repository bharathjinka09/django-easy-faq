========
easy-faq
========

easy-faq is a Django app to allow for a simple yet feature rich faq app. with categories, commenting voting of questions and answers all as an optional part of the app.


Quick start
-----------

1. pip install::

    pip install django-easy-faq

1. Add "easy-faq" to your INSTALLED_APPS setting like this::

    INSTALLED_APPS = [
        ...
        'faq',
    ]

2. Include the easy-faq URLconf in your project urls.py like this::

    path('faq/', include('faq.urls')),


3. Run ``python manage.py makemigrations`` to create the faq models migrations.
4. Run ``python manage.py migrate`` to create the faq models.

5. Start the development server and visit http://127.0.0.1:8000/admin/
   to create a category (you'll need the Admin app enabled).(categories part of the app can be disabled)

6. Visit http://127.0.0.1:8000/faq/ to see the categories.

Settings
--------

you can change most things in settings below is a list of all settings
add any or all to change to desired behavior::


    FAQ_SETTINGS = ['your_settings_here',]


1. no_category_description                  - add if using categories but don't want descriptions for them
2. no_category                              - add if don't want to use categories
3. logged_in_users_can_add_question         - add if you want any logged in user to be able to ask a question
4. logged_in_users_can_answer_question      - add if you want any logged in user to be able to answer a question
5. allow_multiple_answers                   - add if you want a question to be able to be answered multiple times
6. no_comments                              - add if don't want to use comments
7. anonymous_user_can_comment               - add if you want any user to be able to comment including anonymous users
8. view_only_comments                       - add if you want users to see posted comments but not be able to add any more
9. no_votes                                 - add if don't want any voting for useful questions or answers
10. no_answer_votes                         - add if only want question voting
11. no_question_votes                       - add if only want answer voting

Templates
---------

all of the templates are meant to be overwritten
to overwrite them create a faq directory inside of the templates directory and add a html file with the same name to it

if this doesn't work make sure that the templates setting has 'DIRS': ['templates'], in it::

    TEMPLATES = [
        {
            ...
            'DIRS': ['templates'],
            ...
        },
    ]

here is a list of templates and there default template  you can overwrite

1. categories_list.html - faq main view if using categories::

    <h1>select a FAQ category</h1>
    {% for category in categories %}
        <h3><a href="{% url 'faq:category_detail' category.slug %}">{{category.name}}</a></h3>
        {% if category.description %}
            <p>{{category.description}}</p>
        {% endif %}
        <hr>
    {% endfor %}


2. category_detail.html - faq category detail view if using categories::

    <h1>choose a FAQ Question</h1>
    <h2>{{category}}</h2>
    {% if category.description %}
    <p>{{category.description}}</p>
    {% endif %}
    <hr>
    {% for question in category.question_set.all %}
        <h3><a href="{% url 'faq:question_detail' category.slug question.slug %}">{{question.question}}</a></h3>
    {% endfor %}
    <hr>
    <a href="{% url 'faq:index_view' %}">back</a>
    {% if can_add_question %}
        <a href="{% url 'faq:add_question' category.slug %}">add question</a>
    {% endif %}


3. questions_list.html - lists all questions if not using categories::

    <h1>choose a FAQ Question</h1>
    {% for question in questions %}
        <h3><a href="{% url 'faq:question_detail' question.slug %}">{{question.question}}</a></h3>
    {% endfor %}

    {% if can_add_question %}
        <hr>
        <a href="{% url 'faq:add_question' %}">add question</a>
    {% endif %}


4. question_detail.html - the question detail page::

    <h1>{{question.question|title}}</h1>
    {% if can_vote_question %}
        found this question helpful?
        <form style="display: inline;" action="{% if category_enabled %}{% url 'faq:vote_question' question.category.slug question.slug %}{% else %}{% url 'faq:vote_question' question.slug %}{% endif %}" method="post">
            {% csrf_token %}
            <input type="hidden" value=True name="vote">
            <button type="submit">yes({{question.helpful}})</button>
        </form>
        <form style="display: inline;" action="{% if category_enabled %}{% url 'faq:vote_question' question.category.slug question.slug %}{% else %}{% url 'faq:vote_question' question.slug %}{% endif %}" method="post">
            {% csrf_token %}
            <input type="hidden" value=False name="vote">
             <button type="submit">no({{question.not_helpful}})</button>
        </form>
    {% endif %}
    {% if question.category and category_enabled %}
        <p>category - <a href="{% url 'faq:category_detail' question.category.slug %}">{{question.category.name}}</a></p>
    {% endif %}
    <hr>

    {% if allow_multiple_answers %}
    <h3>answers</h3>
    <ul>
        {% for answer in question.answer_set.all %}
            <li><b>{{answer.answer}}</b>
                {% if can_vote_answer %}
                 | found this answer helpful?
                <form style="display: inline;" action="{% if category_enabled %}{% url 'faq:vote_answer' question.category.slug question.slug answer.slug %}{% else %}{% url 'faq:vote_answer' question.slug answer.slug %}{% endif %}" method="post">
                    {% csrf_token %}
                    <input type="hidden" value=True name="vote">
                    <button type="submit">yes({{answer.helpful}})</button>
                </form>
                <form style="display: inline;" action="{% if category_enabled %}{% url 'faq:vote_answer' question.category.slug question.slug answer.slug %}{% else %}{% url 'faq:vote_answer' question.slug answer.slug %}{% endif %}" method="post">
                    {% csrf_token %}
                    <input type="hidden" value=False name="vote">
                    <button type="submit">no({{answer.not_helpful}})</button>
                </form>
                {% endif %}
            </li>
        {% endfor %}
    </ul>

    {% else %}
        {% if question.answer_set.exists %}
            <p>answer:</p>
            <h3>{{question.answer_set.first.answer}}</h3>
            {% if can_vote_answer %}
             found this answer helpful?
            <form style="display: inline;" action="{% if category_enabled %}{% url 'faq:vote_answer' question.category.slug question.slug question.answer_set.first.slug %}{% else %}{% url 'faq:vote_answer' question.slug question.answer_set.first.slug %}{% endif %}" method="post">
                {% csrf_token %}
                <input type="hidden" value=True name="vote">
                <button type="submit">yes({{question.answer_set.first.helpful}})</button>
            </form>
            <form style="display: inline;" action="{% if category_enabled %}{% url 'faq:vote_answer' question.category.slug question.slug question.answer_set.first.slug %}{% else %}{% url 'faq:vote_answer' question.slug question.answer_set.first.slug %}{% endif %}" method="post">
                {% csrf_token %}
                <input type="hidden" value=False name="vote">
                <button type="submit">no({{question.answer_set.first.not_helpful}})</button>
            </form>
            {% endif %}
        {% else %}
            no answers yet
        {% endif %}
    {% endif %}


    {% if can_answer_question %}
        {% if category_enabled %}
            <a href="{% url 'faq:answer_question' question.category.slug question.slug %}">answer question</a>
        {% else %}
            <a href="{% url 'faq:answer_question' question.slug %}">answer question</a>
        {% endif %}
    {% endif %}
    <hr>
    {% if comments_allowed %}
    <h3>comments</h3>
        <ul>
            {% for comment in question.faqcomment_set.all %}
                <li><h4>{{comment.comment}}</h4>
                    posted by {% if comment.user%}{{comment.user}}{% else %}anonymous{% endif %} {{comment.post_time|timesince}} ago</li>
            {% endfor %}
        </ul>
    {% if add_new_comment_allowed %}
        {% if category_enabled %}
        <form method="post" action="{% url 'faq:add_comment' question.category.slug question.slug %}">
        {% else %}
        <form method="post" action="{% url 'faq:add_comment' question.slug %}">

        {% endif %}
        <fieldset>
            <legend>Post Your Comment Here:</legend>
            {% csrf_token %}
            {{comment_form}}
            <input type="submit" name="post">
        </fieldset>
        </form>
        {% endif %}
    {% endif %}

5. answer_form.html - form to add answer to question::

    <h1>Answer Question</h1>
    <a href="{{question.get_absolute_url}}"><h3>{{question.question}}</h3></a>
    <form method="post">
        {% csrf_token %}
        {{form}}
        <input type="submit">
    </form>

6. comment_form.html - form to add comments to question (only shows up when form has error because view only gets posted to)::

    <h1>Post A Comment</h1>
    <a href="{{question.get_absolute_url}}"><h3>{{question.question}}</h3></a>
    <form method="post">
        {% csrf_token %}
        {{form}}
        <input type="submit">
    </form>

7. question_form.html - form to add a new question::

    <h1>Add Your Question</h1>
    <form method="post">
        {% csrf_token %}
        {{form}}
        <input type="submit">
    </form>

8. vote_form.html - form for voting questions and answers (only shows up when form has error because view only gets posted to)::

    <h1>vote</h1>
    <form method="post">
        {% csrf_token %}
        {{form}}
        <input type="submit">
    </form>


Template Variables
------------------
1. categories_list.html
    categories - all the categories (category queryset)

2. categories_detail.html
    category - the category chosen (category object)
    can_add_question - bool if the user can add a question (depends on the settings)
3. questions_list.html
    questions - all the questions (question queryset)
    can_add_question - bool if the user can add a question (depends on the settings)
4. question_detail.html
    question - the question chosen (question object)
    can_vote_question - bool if the user can vote a question (depends on the settings)
    category_enabled - bool if category enabled in settings
    allow_multiple_answers - bool if multiple answers allowed in settings
    can_vote_answer - bool if the user can vote an answer (depends on the settings)
    can_answer_question - bool if current user can answer question (depends on the settings)
    comments_allowed - bool if using comments in settings
    add_new_comment_allowed - bool if current user can add comment (depends on the settings)
    comment_form - form to submit a new comment
5. answer_form.html
    question - the question to add answer to (question object)
    form - form to add new answer
6. comment_form.html
    question - the question to add comment to (question object)
    form - form to add new comment
7. question_form.html
    form - form to add new question
8. vote_form.html
    form - form to vote for a question or answer

change log
----------
0.4 fixed bug that logged out users can vote - which then raises exceptions
0.5 fixed migrations
