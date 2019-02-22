# Insight DevOps Engineering Systems Puzzle

## Troubleshooting Process

Before I started, I installed Docker on my local machine, and run the following commands as describled on the instruction to see what are the issues.

    $ docker-compose up -d db # bring up the database container
    $ docker-compose run --rm flaskapp /bin/bash -c "cd /opt/services/flaskapp/src && python -c  'import database; $ database.init_db()'" # connect to the database container and created the schema
    $ docker-compose up -d # fire up all other services

**Issue 1:** When I open my browser and goes to `localhost:8080`, I see the server is not connected.

**Solve:** I run `docker ps` to see if the services are running and check the port information. Then, I found out that for the nginx container, it's using host port 80 instead of 8080. So, I goes to the docker-compose file and check the mapping port, and I found out that it's the wrong mapping port for the nginx `"80:8080"`, so I reverse it to mapping container port 80 to host port 8080 `8080:80`.

**Issue 2:** I bring down all the containers and remove the volume by running `docker-compose down -v` then start the application again. Now, when I refresh my browser, I can see it's connected to the nginx proxy, but it shows a 502 Bad Gateway error, which means there's something wrong with the origin host, in this case it's our flask app cannot connect to the nginx proxy.

**Solve:** I open the nginx config file `flaskapp.conf` and see if there's any error, and the syntax and logic looks fine to me. But to double check, I found out flask by default using port 5000 instead of 5001 according to its [API documentation](http://flask.pocoo.org/docs/1.0/api/). Then I try change it back to port 5000 and re-run the application again, and I can see the html page! Later when I find out that in the Dockerfile, I also need to change the Expose port from 5001 to 5000.

**Issue 3:** I put some random data to the input fields and hit the button, it redirects me to the success page. The issue now is I don't see my data in the success page.

**Solve:** My first assumption is the data is not storing to the database, but when I try a few times hit the submit button, it's actually seems to storing because the display output changes from `[]` to `[,]`. After sometime going over the code, In `app.py`, I added enabled the debug in `app.run(host='0.0.0.0', debug=True)` and when I run docker-compose, I use `docker-compose up` instead of `docker-compose up -d` so that I can see the debug output. And before returning the `results` I print it. Then I found out it prints object at xxx instead of the value. Then, instead of querying `qry.all()`, I used `qry.first()` and try print it out, it also prints out  `<models.Items object at 0x7f955d1f5f98>`. Because it's object, it should have property, and the properties are name, quantity, description, date_added, so I try `qry.first().name` and it actually prints out the name I input on the page as I expected. Now, I can loop through all rows when I use `qry.all()` and make a `one_item` dictionary to hold a row's data and append each row to the results array. At the end, when I type some input and hit the button, it redirects me to the success page with all my input data showing up!
