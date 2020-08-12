# Remote database challenge

Practice deploying a PostgreSQL database to Heroku, plus some advanced SQL commands.

## Remote databases

You don't want your deployed production app talking to a database running on your laptop. This would be slow, insecure and require you to leave it turned on all the time.

Instead we can host our production database on a 3rd party service like Heroku.

Follow these [instructions for setting up a Heroku database](https://dev.to/prisma/how-to-setup-a-free-postgresql-database-on-heroku-1dc1).

Once you're done you should have a connection string that looks something like this:

```sh
postgres://okaws:d3fa@ec2-54.eu-west-1.compute.amazonaws.com:5432/d8bvo
```

You can connect to the remote database from your terminal by running:

```sh
psql your_url_goes_here
```

## Advanced SQL

Let's practice some more advanced SQL commands. You will need to read `init.sql` to see what values each table contains.

You may have to search the internet for commands you haven't seen before. [W3 Schools](https://www.w3schools.com/sql/) is a good resource.

### Setup

1. Clone this repo
1. Connect to the database with `psql your_database_url`
1. Insert some example data with `\i init.sql`

You can check everything is set up by listing the database tables with `\dt`. You should see four FAC-related tables: `cohorts`, `students`, `projects` and `students_projects`.

1.  ### Cohort locations

    List the names of all cohorts that took place in Finsbury Park.

    #### Expected result

    | name |
    | ---- |
    | 14   |
    | 15   |
    | 16   |
    | 17   |

    <details>
    <summary>Solution</summary>

    ```sql
    SELECT name FROM cohorts WHERE location = 'Finsbury Park';
    ```

    </details>

1.  ### Student locations

    List the usernames of all students who attended FAC in Finsbury Park.

    <details>
    <summary>Hint</summary>
    You need to use the query from the previous question.
    </details>

    #### Expected result

    | username       |
    | -------------- |
    | virtualdominic |
    | charlielafosse |
    | starsuit       |
    | bobbysebolao   |
    | albadylic      |
    | reubengt       |

    <details>
    <summary>Solution</summary>

    ```sql
    SELECT username FROM students WHERE cohort_name IN (
      SELECT name FROM cohorts WHERE location = 'Finsbury Park'
    );
    ```

    </details>

1.  ### Student locations

    List the username of each student along with the location of their cohort.

    <details>
    <summary>Hint</summary>
    Remember you can use joins to connect two tables together and access information from both.
    </details>

    #### Expected result

    | username       | location      |
    | -------------- | ------------- |
    | eliascodes     | Bethnal Green |
    | oliverjam      | Bethnal Green |
    | yvonne-liu     | Bethnal Green |
    | matthewdking   | Nazareth      |
    | helenzhou6     | Bethnal Green |
    | virtualdominic | Finsbury Park |
    | charlielafosse | Finsbury Park |
    | starsuit       | Finsbury Park |
    | bobbysebolao   | Finsbury Park |
    | albadylic      | Finsbury Park |
    | reubengt       | Finsbury Park |

    <details>
    <summary>Solution</summary>

    ```sql
    SELECT students.username, cohorts.location FROM students
      INNER JOIN cohorts ON students.cohort_name = cohorts.name;
    ```

    </details>

1.  ### Students with projects

    List all project names with the usernames of the students who worked on them.

    <details>
    <summary>Hint</summary>

    Since projects-to-students is a _many-to-many_ relationship (each project can have multiple authors, each student can have multiple projects) we can't link them with just IDs. We need a whole separate table to keep track of which students worked on which projects.

    This is often called a _join table_, or _junction table_. You'll need to join to this as an intermediary step to link projects to students.

    </details>

    #### Expected result

    | name          | username     |
    | ------------- | ------------ |
    | FACX Machine  | oliverjam    |
    | FACX Machine  | yvonne-liu   |
    | Hamster Hotel | oliverjam    |
    | Hamster Hotel | starsuit     |
    | Agony Yaunt   | starsuit     |
    | Agony Yaunt   | bobbysebolao |

    <details>
    <summary>Solution</summary>

    ```sql
    SELECT projects.name, students.username FROM projects
      INNER JOIN students_projects ON projects.id = students_projects.project_id
      INNER JOIN students ON students.username = students_projects.student_username;
    ```

    </details>

1.  ### Bonus: Students with projects by location

    List all project names with the usernames of the students who worked on them, only for students who attended FAC in Finsbury Park.

    <details>
    <summary>Hint</summary>
    You've written all the queries you need in previous steps.
    </details>

    #### Expected result

    | name          | username     |
    | ------------- | ------------ |
    | Hamster Hotel | starsuit     |
    | Agony Yaunt   | starsuit     |
    | Agony Yaunt   | bobbysebolao |

    <details>
    <summary>Solution</summary>

    ```sql
    SELECT projects.name, students.username FROM projects
      INNER JOIN students_projects ON projects.id = students_projects.project_id
      INNER JOIN students ON students.username = students_projects.student_username
      WHERE students.username IN (
        SELECT username FROM students
          WHERE cohort_name IN (
            SELECT name FROM cohorts WHERE location = 'Finsbury Park'
          )
      );
    ```

    </details>
