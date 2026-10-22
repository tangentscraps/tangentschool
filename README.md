# tangentscraps/tangentschool

> Software to run an online course: a syllabus rendered as a prerequisite graph,
> step-based lessons with quizzes, and progress tracking.
> Repo: https://github.com/tangentscraps/tangentschool — ~22 files. Last pushed 2016-04-06.

## What it was

**Tangentschool** — a from-scratch online-course platform. The syllabus is a directed
graph of lessons drawn with a d3 force layout: lessons are nodes, prerequisites are
edges, and nodes are color-coded by completion state. A lesson unlocks only once its
prerequisites are done. Inside a lesson, content is a sequence of *steps* driven by a
tiny opcode set: `___` (new screen), `say` (display text), `ask` (question textbox),
`chk` (check the answer against a key, with pass/fail messages). The backend stores
every answer attempt (for analytics) plus per-user lesson progress. Login was
federated via Auth0.

The seed data is an "Intro" course in digital logic: bits → bit vectors →
truth tables → XOR/AND → half adder → counter → register → RAM → fetch →
instruction pointer. So the first (and only real) course was computer-hardware
fundamentals.

## Tech stack

- **Backend:** Python WSGI stub (eventlet + psycopg2), four JSON endpoints:
  `/courses`, `/lessons` (d3-format graph), `/steps`, `/check` (answer checking)
- **Database:** PostgreSQL 9.5 — `course`, `lesson`, `prereq`, `step`, `opcode`,
  `user`, `user_lesson`, `user_answer` tables plus a `course_lessons` view
- **Frontend:** Angular 2 (beta.13) + TypeScript (not CoffeeScript — the README's
  claim is stale), d3 force-layout graph directive, angular2-jwt, Auth0Lock
- **Deployment:** nginx + WSGI on Ubuntu 15.10 (full manual recipe in the README)

## Inventory

| Path | Contents |
|------|----------|
| `README.md` | Project description + Ubuntu 15.10 deployment recipe (nginx, node 5.x, PostgreSQL 9.5) |
| `app/serve.py` | The whole backend: WSGI dispatch, 4 JSON endpoints, answer checking |
| `app/tangentschool/__init__.py` | Empty package init |
| `app/requirements.txt` | eventlet, psycopg2 |
| `sql/tangentschool.sql` | Schema + "Intro" course seed data (digital-logic lesson graph, step opcodes, a sample bits lesson) |
| `web/index.html` | App shell |
| `web/src/school.ts` | Root component, routing, Auth0 login/logout |
| `web/src/courses.ts` | Course list |
| `web/src/lesson.ts` | Lesson view |
| `web/src/stage.ts` | Step-by-step lesson stage |
| `web/src/graph.ts` | d3 force-layout syllabus graph (color-coded by done bit, locked until prereqs met) |
| `web/style/school.css` | 6 lines of CSS |
| `etc/nginx/` | nginx config for the deployment |
| `LICENSE` | MIT, 2016 tangentstorm |

## Surprising / notable

- **Stale README:** it says the frontend is "html/css/coffee-script" but the code is
  TypeScript/Angular 2 — the README predates a frontend rewrite.
- **Prototype-grade backend:** hardcoded `uid = 1`, hardcoded lesson name `'bits'`
  with `# TODO: fetch from url` comments, and a literal `<redacted>` placeholder
  password in `serve.py` — this never ran as a real service.
- **Waffle.io kanban badge** in the README — Waffle shut down years ago.
- No linked streams or videos found anywhere in the repo.
- The commit history shows the real feature arc: basic routing → lesson steps →
  answer checking → d3 graph → prerequisite unlocking → Auth0 federated login.

---

## Original README (deployment notes, pre-curation)

Project kanban at waffle.io: [![Stories in Ready](https://badge.waffle.io/tangentstorm/tangentschool.png?label=ready&title=Ready)](https://waffle.io/tangentstorm/tangentschool)

# tangentschool
software to run an online course

- **app** contains the python backend
- **web** contains the html/css/coffee-script frontend

# deployment (ubuntu 15.10)

First, install the default nginx:

    sudo apt-get install nginx

Copy files to:

| local | remote          |
|-------|-----------------|
| etc/* | /etc/*          |
| web/* | /var/www/html/* |
| app/* | /var/www/wsgi/* |


Update nginx to use the new config file:

    cd /etc/nginx/sites-enabled/
    sudo rm default
    sudo ln -s ../sites-available/tangentschool.conf
    sudo service nginx restart


install node 5.x for the front-end:

    curl -sL https://deb.nodesource.com/setup_5.x | sudo -E bash -
    sudo apt-get install -y nodejs
    cd /var/www/html
    npm install

Install PostgreSQL 9.5:

    sudo su -
    echo 'deb http://apt.postgresql.org/pub/repos/apt/ wily-pgdg main' \
        >  /etc/apt/sources.list.d/pgdg.list
    wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc \
        | apt-key add -
    apt-get update
    sudo apt-get install postgresql-9.5 postgresql-server-dev-9.5
    
    # make sure the 'port' line is set to '5432' (not 5433):
    grep ^port  /etc/postgresql/9.5/main/postgresql.conf
    # (fix if necessary)
     
    service postgresql restart
    exit

Create a user and database:

    sudo su - postgres
    createuser -P tangentschool
    # (enter password twice)
    createdb -O tangentschool tangentschool
    exit

add some data:

    psql -h localhost -U tangentschool
    # <enter password>
    \i /path/to/tangentschool/sql/tangentschool.sql


now set up the python backend:

    mkdir /var/www/wsgi
    cd /var/www/wsgi
    sudo apt-get install -y python-pip python-dev
    pip install -r requirements.txt
    python serve.py

