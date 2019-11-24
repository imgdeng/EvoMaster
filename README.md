# EvoMaster: A Tool For Automatically Generating System-Level Test Cases


![](docs/img/carl-cerstrand-136810_compressed.jpg  "Photo by Carl Cerstrand on Unsplash")

[![Build Status](https://travis-ci.org/EMResearch/EvoMaster.svg?branch=master)](https://travis-ci.org/EMResearch/EvoMaster)
[![CircleCI](https://circleci.com/gh/EMResearch/EvoMaster.svg?style=svg)](https://circleci.com/gh/EMResearch/EvoMaster)
[![Maven Central](https://maven-badges.herokuapp.com/maven-central/org.evomaster/evomaster-client-java/badge.svg)](https://maven-badges.herokuapp.com/maven-central/org.evomaster/evomaster-client-java)
<!---
Needs auth :(
[[JaCoCo]](https://circleci.com/api/v1.1/project/github/arcuri82/evomaster/latest/artifacts/0/home/circleci/evomaster-build/report/target/site/jacoco-aggregate/index.html)
-->

<!--
<b>HIRING</b>: we are hiring postdocs to work on this project.
For more information and for applying, see [here]().
<br>
-->

_EvoMaster_ ([www.evomaster.org](http://evomaster.org)) is a tool prototype 
that automatically *generates* system-level test cases.
Internally, it uses an [Evolutionary Algorithm](https://en.wikipedia.org/wiki/Evolutionary_algorithm) 
and [Dynamic Program Analysis](https://en.wikipedia.org/wiki/Dynamic_program_analysis)  to be 
able to generate effective test cases.
The approach is to *evolve* test cases from an initial population of 
random ones, trying to maximize measures like code coverage and fault detection.


At the moment, _EvoMaster_ targets RESTful APIs compiled to 
JVM 8 bytecode.
The APIs must provide a schema in [OpenAPI/Swagger](https://swagger.io) format.
The tool generates JUnit (version 4 or 5) tests, written in either Java or Kotlin.


A short [video](https://youtu.be/7zTLUlH-BNI) shows the use of _EvoMaster_ on one of the 
case studies in [EMB](https://github.com/EMResearch/EMB). 


_EvoMaster_ is currently (2018-2021) funded by a 7.8 million Norwegian Kroner grant 
by the Research Council of Norway (RCN), as part of the 
Frinatek project <i>Evolutionary Enterprise Testing</i>.  




### Examples

![](docs/img/evomaster_console.png)


The following code is an example of one test that was automatically
generated by _EvoMaster_ for a REST service called 
"scout-api" (see [EMB repository](https://github.com/EMResearch/EMB)).
The generated test uses the [RestAssured](https://github.com/rest-assured/rest-assured) library.

```
@Test
public void test_36_with500() throws Exception {
        
   String location_media_files = "";
        
   String id_0 = given().accept("application/json")
                .header("Authorization", "ApiKey moderator") // moderator
                .contentType("application/json")
                .body(" { " + 
                    " \"uri\": \"hXf3e8B3ikGtuGjT\", " + 
                    " \"name\": \"nkeUfXVTC\", " + 
                    " \"copy_right\": \"\" " + 
                    " } ")
                .post(baseUrlOfSut + "/api/v1/media_files")
                .then()
                .statusCode(200)
                .assertThat()
                .contentType("application/json")
                .body("'uri'", containsString("hXf3e8B3ikGtuGjT"))
                .body("'name'", containsString("nkeUfXVTC"))
                .body("'copy_right'", containsString(""))
                .extract().body().path("id").toString();
                
   location_media_files = "/api/v1/media_files/" + id_0;
        
   given().accept("*/*")
                .header("Authorization", "ApiKey moderator") // moderator
                .get(resolveLocation(location_media_files, baseUrlOfSut + "/api/v1/media_files/1579038228/file"))
                .then()
                .statusCode(500) // se/devscout/scoutapi/resource/MediaFileResource_268_downloadFile
                .assertThat()
                .contentType("application/json")
                .body("'code'", numberMatches(500.0));
}
```

In this automatically generated test, a new resource is first created with a _POST_ command.
The _id_ of this newly generated resource is then extracted from the _POST_ response, and used in the URL
of a following _GET_ request on a sub-resource.
Such _GET_ request does break the backend, as it returns a __500__ HTTP status code.
The last line executed in the backend is then printed as comment, to help debugging this fault.    


The generated tests are self-contained, i.e., they 
start/stop the REST server by themselves:

```
    private static SutHandler controller = new em.embedded.se.devscout.scoutapi.EmbeddedEvoMasterController();
    private static String baseUrlOfSut;
    
    
    @BeforeClass
    public static void initClass() {
        baseUrlOfSut = controller.startSut();
        assertNotNull(baseUrlOfSut);
        RestAssured.urlEncodingEnabled = false;
    }
    
    
    @AfterClass
    public static void tearDown() {
        controller.stopSut();
    }
    
    
    @Before
    public void initTest() {
        controller.resetStateOfSUT();
    }
```

The ability of starting/resetting/stopping the tested application is critical for using the generated 
tests in _Continuous Integration_ (e.g., Jenkins, Travis and CircleCI).
However, it requires to write a [_Driver_](docs/write_driver.md) to tell _EvoMaster_ how to do 
such start/reset/stop.
  

TODO SQL



### Documentation


* [Download EvoMaster](docs/download.md)
* [Build EvoMaster from source](docs/build.md)
* [Main console options](docs/options_main.md)
    * [All console options](docs/options_all.md)
* [Using EvoMaster for Black-Box Testing (easier to setup, but worse results)](docs/blackbox.md)
* [Using EvoMaster for White-Box Testing (harder to setup, but better results)](docs/whitebox.md)
    * [Write an EvoMaster Driver for White-Box Testing](docs/write_driver.md)
* [Academic papers related to EvoMaster](docs/publications.md)
* [Slides of presentations/seminars](docs/presentations.md)
* [Notes for developers contributing to EvoMaster](docs/for_developers.md)



### How to Contribute

There are many ways in which you can contribute.
If you found _EvoMaster_ of any use, _the easiest
way to show appreciation is to **star** it_.
Issues and feature requests can be reported on
the [issues](https://github.com/EMResearch/EvoMaster/issues) page:  
  
* *Bugs*: as for any bug report, the more detailed
  you can be the better.
  If you are using _EvoMaster_ on an open source project,
  please provide links to it, as then it is much easier
  to reproduce the bugs.
  
* If you are trying to use _EvoMaster_, but the instructions
  in these notes are not enough to get you started, 
  then it means it is a "bug" in the documentation, which then would need
  to be clarified. 
  
* *Feature Requests*: to improve _EvoMaster_,
  we are very keen to receive
  feature requests, although of course we cannot
  guarantee when they are going to be implemented. 
  
* *Pull Requests*: we are very keen to receive PRs, as long as you agree
  with the license of _EvoMaster_. However, before making a PR, should read
  the [notes for developers](docs/for_developers.md).  
    



### License
_EvoMaster_'s source code is released under the LGPL (v3) license.
For a list of the used third-party libraries, you can directly see the root [pom.xml](./pom.xml) file.
For a list of code directly imported (and then possibly modified/updated) from 
other open-source projects, see [here](./docs/reused_code.md).


### ![](https://www.yourkit.com/images/yklogo.png)

YourKit supports open source projects with its full-featured Java Profiler.
YourKit, LLC is the creator of 
<a href="https://www.yourkit.com/java/profiler/">YourKit Java Profiler</a>
and 
<a href="https://www.yourkit.com/.net/profiler/">YourKit .NET Profiler</a>,
innovative and intelligent tools for profiling Java and .NET applications.


