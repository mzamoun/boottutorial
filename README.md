# boottutorial

Test the application

Now that the application is running, you can test it. You can use any REST client you wish. The following examples use the *nix tool curl.

First you want to see the top level service.

$ curl http://localhost:8080
{
  "_links" : {
    "people" : {
      "href" : "http://localhost:8080/people{?page,size,sort}",
      "templated" : true
    }
  }
}
Here you get a first glimpse of what this server has to offer. There is a people link located at http://localhost:8080/people. It has some options such as ?page, ?size, and ?sort.

Spring Data REST uses the HAL format for JSON output. It is flexible and offers a convenient way to supply links adjacent to the data that is served.
$ curl http://localhost:8080/people
{
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/people{?page,size,sort}",
      "templated" : true
    },
    "search" : {
      "href" : "http://localhost:8080/people/search"
    }
  },
  "page" : {
    "size" : 20,
    "totalElements" : 0,
    "totalPages" : 0,
    "number" : 0
  }
}
There are currently no elements and hence no pages. Time to create a new Person!

$ curl -i -X POST -H "Content-Type:application/json" -d '{  "firstName" : "Frodo",  "lastName" : "Baggins" }' http://localhost:8080/people
HTTP/1.1 201 Created
Server: Apache-Coyote/1.1
Location: http://localhost:8080/people/1
Content-Length: 0
Date: Wed, 26 Feb 2014 20:26:55 GMT
-i ensures you can see the response message including the headers. The URI of the newly created Person is shown
-X POST signals this a POST used to create a new entry
-H "Content-Type:application/json" sets the content type so the application knows the payload contains a JSON object
-d '{ "firstName" : "Frodo", "lastName" : "Baggins" }' is the data being sent
Notice how the previous POST operation includes a Location header. This contains the URI of the newly created resource. Spring Data REST also has two methods on RepositoryRestConfiguration.setReturnBodyOnCreate(…) and setReturnBodyOnCreate(…) which you can use to configure the framework to immediately return the representation of the resource just created.
From this you can query for all people:

$ curl http://localhost:8080/people
{
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/people{?page,size,sort}",
      "templated" : true
    },
    "search" : {
      "href" : "http://localhost:8080/people/search"
    }
  },
  "_embedded" : {
    "persons" : [ {
      "firstName" : "Frodo",
      "lastName" : "Baggins",
      "_links" : {
        "self" : {
          "href" : "http://localhost:8080/people/1"
        }
      }
    } ]
  },
  "page" : {
    "size" : 20,
    "totalElements" : 1,
    "totalPages" : 1,
    "number" : 0
  }
}
The persons object contains a list with Frodo. Notice how it includes a self link. Spring Data REST also uses Evo Inflector to pluralize the name of the entity for groupings.

You can query directly for the individual record:

$ curl http://localhost:8080/people/1
{
  "firstName" : "Frodo",
  "lastName" : "Baggins",
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/people/1"
    }
  }
}
This might appear to be purely web based, but behind the scenes, there is an H2 relational database. In production, you would probably use a real one, like PostgreSQL.
In this guide, there is only one domain object. With a more complex system where domain objects are related to each other, Spring Data REST will render additional links to help navigate to connected records.

Find all the custom queries:

$ curl http://localhost:8080/people/search
{
  "_links" : {
    "findByLastName" : {
      "href" : "http://localhost:8080/people/search/findByLastName{?name}",
      "templated" : true
    }
  }
}
You can see the URL for the query including the HTTP query parameter name. If you’ll notice, this matches the @Param("name") annotation embedded in the interface.

To use the findByLastName query, do this:

$ curl http://localhost:8080/people/search/findByLastName?name=Baggins
{
  "_embedded" : {
    "persons" : [ {
      "firstName" : "Frodo",
      "lastName" : "Baggins",
      "_links" : {
        "self" : {
          "href" : "http://localhost:8080/people/1"
        }
      }
    } ]
  }
}
Because you defined it to return List<Person> in the code, it will return all of the results. If you had defined it only return Person, it will pick one of the Person objects to return. Since this can be unpredictable, you probably don’t want to do that for queries that can return multiple entries.

You can also issue PUT, PATCH, and DELETE REST calls to either replace, update, or delete existing records.

$ curl -X PUT -H "Content-Type:application/json" -d '{ "firstName": "Bilbo", "lastName": "Baggins" }' http://localhost:8080/people/1
$ curl http://localhost:8080/people/1
{
  "firstName" : "Bilbo",
  "lastName" : "Baggins",
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/people/1"
    }
  }
}
$ curl -X PATCH -H "Content-Type:application/json" -d '{ "firstName": "Bilbo Jr." }' http://localhost:8080/people/1
$ curl http://localhost:8080/people/1
{
  "firstName" : "Bilbo Jr.",
  "lastName" : "Baggins",
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/people/1"
    }
  }
}
PUT replaces an entire record. Fields not supplied will be replaced with null. PATCH can be used to update a subset of items.
You can delete records:

$ curl -X DELETE http://localhost:8080/people/1
$ curl http://localhost:8080/people
{
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/people{?page,size,sort}",
      "templated" : true
    },
    "search" : {
      "href" : "http://localhost:8080/people/search"
    }
  },
  "page" : {
    "size" : 20,
    "totalElements" : 0,
    "totalPages" : 0,
    "number" : 0
  }
}

