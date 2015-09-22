[Source](http://www.anthonydebarros.com/2012/03/11/generate-json-from-sql-using-python/ "Permalink to Generate JSON From SQL Using Python")

# Generate JSON From SQL Using Python

Let's say you want to generate a few hundred — or even a thousand — flat JSON files from a SQL database. Maybe you want to power an interactive graphic but have neither the time nor the desire to spin up a server to dynamically generate the data. Or you think a server adds one more piece of unnecessary complexity and administrative headache. So, you want flat files, each one small for quick loading. And a lot of them.

A few lines of Python is all you need.

I've gone this route lately for a few data-driven interactives at USA TODAY, creating JSON files out of large data sets living in SQL Server. Python works well for this, with its [JSON encoder/decoder][1] offering a flexible set of tools for converting Python objects to JSON.

Here's a brief tutorial:

1\. If you haven't already, install Python. Here's my guide to [setup on Windows 7][2]; if you're on Linux or Mac you should have it already.

2\. In your Python script, import a database connector. This example uses [pyodbc][3], which supports connections to SQL Server, MySQL, Microsoft Access and other databases. If you're using PostgreSQL, try [psycopg2][4].

3\. Create a table or tables to query in your SQL database and write and test your query. In this example, I have a table called Students that has a few fields for each student. The query is simple:  
&nbsp;

| ----- |
|

    SELECT ID, FirstName, LastName, Street, City, ST, Zip
    FROM Students

 |

4\. Here's an example script that generates two JSON files from that query. One file contains JSON row arrays, and the other JSON key-value objects. Below, we'll walk through it step-by-step.  
  
&nbsp;

| ----- |
|

    import pyodbc
    import json
    import collections
    &nbsp;
    connstr = 'DRIVER={SQL Server};SERVER=ServerName;DATABASE=Test;'
    conn = pyodbc.connect(connstr)
    cursor = conn.cursor()
    &nbsp;
    cursor.execute("""
                SELECT ID, FirstName, LastName, Street, City, ST, Zip
                FROM Students
                """)
    &nbsp;
    rows = cursor.fetchall()
    &nbsp;
    # Convert query to row arrays
    &nbsp;
    rowarray_list = []
    for row in rows:
        t = (row.ID, row.FirstName, row.LastName, row.Street,
             row.City, row.ST, row.Zip)
        rowarray_list.append(t)
    &nbsp;
    j = json.dumps(rowarray_list)
    rowarrays_file = 'student_rowarrays.js'
    f = open(rowarrays_file,'w')
    print &gt;&gt; f, j
    &nbsp;
    # Convert query to objects of key-value pairs
    &nbsp;
    objects_list = []
    for row in rows:
        d = collections.OrderedDict()
        d['id'] = row.ID
        d['FirstName'] = row.FirstName
        d['LastName'] = row.LastName
        d['Street'] = row.Street
        d['City'] = row.City
        d['ST'] = row.ST
        d['Zip'] = row.Zip
        objects_list.append(d)
    &nbsp;
    j = json.dumps(objects_list)
    objects_file = 'student_objects.js'
    f = open(objects_file,'w')
    print &gt;&gt; f, j
    &nbsp;
    conn.close()

 |

&nbsp;  
Let's break this down. After our import statements, we set a connection string to the server. Then, we use pyodbc to open that connection and execute the query:  
&nbsp;

| ----- |
|

    connstr = 'DRIVER={SQL Server};SERVER=ServerName;DATABASE=Test;'
    conn = pyodbc.connect(connstr)
    cursor = conn.cursor()
    &nbsp;
    cursor.execute("""
                SELECT ID, FirstName, LastName, Street, City, ST, Zip
                FROM Students
                """)
    &nbsp;
    rows = cursor.fetchall()

 |

The script loads the query results into a list object called rows, which we can iterate through to do any number of things. In this case, we'll build JSON.

At the top of the file, the script imports Python's json module, which translates Python objects to JSON and vice-versa. Python lists and tuples become arrays while dictionaries become objects with key-value pairs.

In the first example, the script builds a list of tuples, with each row in the database becoming one tuple. Then, the json module's "dumps" method is used to serialize the list of tuples to JSON, and we write to a file:  
&nbsp;

| ----- |
|

    rowarray_list = []
    for row in rows:
        t = (row.ID, row.FirstName, row.LastName, row.Street,
             row.City, row.ST, row.Zip)
        rowarray_list.append(t)
    &nbsp;
    j = json.dumps(rowarray_list)
    rowarrays_file = 'student_rowarrays.js'
    f = open(rowarrays_file,'w')
    print &gt;&gt; f, j

 |

The JSON result looks like this, with each tuple in a JSON array:  
&nbsp;

| ----- |
|

    [
        [
            1,
            "Samantha",
            "Baker",
            "9 Main St.",
            "Hyde Park",
            "NY",
            "12538"
        ],
        [
            2,
            "Mark",
            "Salomon",
            "12 Destination Blvd.",
            "Highland",
            "NY",
            "12528"
        ]
    ]

 |

That validates nicely over at [JSONLint.com][5].

As a second example, the script next builds a list of dictionaries, with each row in the database becoming one dictionary and each field in the row a key-value pair:  
&nbsp;

| ----- |
|

    objects_list = []
    for row in rows:
        d = collections.OrderedDict()
        d['id'] = row.ID
        d['FirstName'] = row.FirstName
        d['LastName'] = row.LastName
        d['Street'] = row.Street
        d['City'] = row.City
        d['ST'] = row.ST
        d['Zip'] = row.Zip
        objects_list.append(d)
    &nbsp;
    j = json.dumps(objects_list)
    objects_file = 'student_objects.js'
    f = open(objects_file,'w')
    print &gt;&gt; f, j

 |

You'll note that I'm using Python's [OrderedDict() object][6] from its collections class in place of a regular dictionary. While this is not necessary, I like to use it so I can force the order of dictionary keys to make the JSON more readable for fact-checking. Be sure to import collections at the beginning of your script.

Here's the resulting JSON in valid objects:  
&nbsp;

| ----- |
|

    [
        {
            "id": 1,
            "FirstName": "Samantha",
            "LastName": "Baker",
            "Street": "9 Main St.",
            "City": "Hyde Park",
            "ST": "NY",
            "Zip": "12538"
        },
        {
            "id": 2,
            "FirstName": "Mark",
            "LastName": "Salomon",
            "Street": "12 Destination Blvd.",
            "City": "Highland",
            "ST": "NY",
            "Zip": "12528"
        }
    ]

 |

That's it.

Using these simple building blocks, you can now construct basic or complex JSON output for one or one gazillion files. For example:

— Use pyodbc to query a list of students, then iterate over that list to output a single file on each student. This would be handy if you had a lot of data on each student and you wanted to keep each JSON file small for quick loading.  
— You can add nested objects by executing queries for related records, building them into dictionaries and appending them to the output.

Very handy, and it runs quickly to boot.

[1]: http://docs.python.org/library/json.html
[2]: http://www.anthonydebarros.com/2011/10/15/setting-up-python-in-windows-7/
[3]: http://code.google.com/p/pyodbc/
[4]: http://initd.org/psycopg/
[5]: http://http://jsonlint.com/
[6]: http://docs.python.org/library/collections.html#ordereddict-objects
