---
title: "Practical TDD Using Ktor"
date: 2022-02-18T09:57:09+13:00
draft: false
description: "Using TDD to create a web API in Kotlin using the Ktor framework"
tags: [
"ktor",
"kotlin",
"testing",
"TDD"
]
---
## Preamble

I have always wanted to get into blogging and finally, the time has come. I am sitting here writing my first ever article, and it all happened
because of a YouTube comment. The comment was only meant to be a few paragraphs long, but it eventually turned into a crazy long post. [Check it out here if you are interested](https://www.youtube.com/watch?v=yfP_v6qCdcs&lc=UgyYduHcMnCzMkIF8b94AaABAg.9S3LUd9Fgtv9S3ngtWyran)
(you'll need to scroll down to the comment section to see it).

I have never felt the need to leave a comment on anything online because I feel like I am not contributing
anything of value, and I do not want to pollute the already difficult to navigate comment sections with more noise. However, in this instance, I
felt the urge to write a comment.

> This kind of content is great! Getting started with TDD is hard.
> 
> I'd love to pay for a TDD course which focuses less on the classical katas and more on backend web development.
> What happens when we have a /fizzbuzz POST endpoint and save the result to a database, for example?

This was the original comment I was replying to. Something really clicked when I read that comment.
Perhaps it was because I saw a younger version of myself when reading the comment who was also in the same position a few years ago. Perhaps it was because
so many examples online do not reflect the reality of software development. I am tired of reading the documentation for new frameworks,
programming languages, tools, or development paradigms that have bad examples which do not reflect real-world usage.
Recently I ran into this exact problem. I was trying to understand how dependency injection worked in a particular framework
[I am looking at you Micronaut](https://docs.micronaut.io/latest/guide/#ioc). Their example of a vehicle and engines just added more
complexity because I had to translate their analogy and apply it to what I was working on (creating a web API using the MVC pattern).
I already had an understanding of dependency injection, so I did not need an analogy. I was not building a car, I was building a web API, I was using
a web framework to make a web API, why are the example not related to common web API usages and patterns?
If I didn't understand dependency injection then I would read up about it and come back to the documentation once I was ready.

Don't get me wrong, I do like the micronaut framework, but their docs just drive me nuts at times. I think this may have been why I felt so
compelled to reply to that comment. I could feel the frustration that they felt, and I wanted to do something about it.
Anyway, here is my attempt at sharing (hopefully) some practical knowledge without the added bullshit.
I hope you enjoy it, but more importantly, I hope that you can take something away from this.

## Prerequisites

This article assumes you have some prior experience working with web apis and have an understanding of the following concepts:
- Have a basic understanding of TDD
- Have a basic understanding of modern web frameworks that follow the MVC pattern
- Have a basic understanding of dependency injection
- Have a basic understanding of mocking libraries
- Have a REST client that you are comfortable with such as Postman, curl, or Insomnia

## Code Along

It is not necessary to code along to get benefit from this article, but if you wish to do so follow these steps to get started:

- Before getting started if you want to code along instead of just reading you can [clone the repo from GitHub](https://github.com/kirillov22/ktor-tdd)
- Read the readme in the project on how to get started if you want to code along. It is best if you start from [here](https://github.com/kirillov22/ktor-tdd/tree/7c7a04197e2a0f655de104e5b9bb862e417cfcb7)

You can switch to that commit which has the starting code with the following command:

`git checkout 7c7a04197e2a0f655de104e5b9bb862e417cfcb7`

## Project Brief

In this project, we will build a simple RESTFul CRUD (create, read, update and delete) API written in Kotlin using the Ktor framework.
I have chosen to create an API for managing student information at a fictitious university. This is because it is a realistic example of
something that you would build in the real world. Here is a rough breakdown of how it should work.

- It will return information about student enrollments and student grades.
- It will follow the MVC pattern where we have a controller that defines the endpoints, parses requests and returns responses, a service class
that will handle the business logic of the app, and a repository class that will handle data storage and persistence.
- Finally, we will also be using the Kodein library for dependency injection, MockK for mocking, and Kotlin-test for unit testing.

Why did I choose to use these technologies? I have previously built apis using C#, NodeJS, and Java, but I have yet to build a full-blown Kotlin REST API.
This seemed like a perfect opportunity for me to try it out, learn something new and see how it compares to the technologies I have used.

## Project Structure

![Project Structure](/images/ktor-tdd-project-structure.png "Project Structure")

We have a few sub-directories in our project that organise the files in a way that you would normally see in an enterprise environment.
I will briefly explain what they are all for, but I encourage you to have a look and get familiar with the files.

- `Application`code - This is the main entry point into our application. We will be performing the dependency injection here, in a more complex
  application it might be a good idea to separate the dependency injection into its own file, but it is ok for now as we have a simple application.
- `controller` - This is where the endpoints and routes are defined. We will be handling the requests and responses here.
- `model` - This is where we define the model/ data classes in our application.
- `repository` - This is where we will handle the data persistence in our application.
- `service` - This is where we will handle the bulk of the business logic of our application.
- `test` - This is where we will have all of our tests. The test directory should reflect the main code structure, so it is easier to follow.
  We will be adding more test files and directories as we go.

## Let's Get Coding

I have included a simple response for one of the endpoints to test that everything is set up and working correctly. Run the app and make a GET request
either in your browser or some other client to `localhost:8080/students` and see if you get the following response back.

```json
[{"id":1,"name":"bill","dateOfBirth":{"year":1990,"month":3,"day":4},"enrolledClasses":[],"averageGpa":0.1}]
```

If that works then go ahead and replace the code for the student endpoint with `TODO()`. We are going to do proper TDD where we write the tests first 
and then the actual code to pass the tests.

## TDD For the Controller

Once that is all working let's start in true TDD fashion and run the tests. We should see that the test fails. That's ok we are going to fix it.
However, before we do that let's actually put the test in a better location. Create a new directory called `controller` inside the main
test directory with a file called `StudentControllerTestCase`.

## GET /students

In that class, we will add the following code which will be testing the `/students` endpoint.

```kotlin
class StudentControllerTestCase {

    @MockK
    private lateinit var studentRepository: StudentRepository

    @MockK
    private lateinit var studentService: StudentService

    @Before
    fun setUp() {
        MockKAnnotations.init(this)
        every { studentService.getStudents() } returns getTestStudents()
    }

    @Test
    fun `should return all students when GET request is made to students endpoint`() {
        withTestApplication(configure()) {
            handleRequest(HttpMethod.Get, "/students").apply {
                assertThat(response.status()).isEqualTo(HttpStatusCode.OK)
                val expectedStudents = getTestStudents()
                val rawResponse = response.content ?: ""
                val actualStudents = Json.decodeFromString<List<Student>>(rawResponse)
                assertThat(actualStudents).isEqualTo(expectedStudents)
            }
        }
    }

    private fun configure(): Application.() -> Unit = {
        install(ContentNegotiation) {
            json()
        }
        di {
            bind { singleton { studentRepository }}
            bind { singleton { studentService } }
        }
        configureRouting()
    }

    private fun getTestStudents(): List<Student> {
        val birthDate1 = LocalDate(1990, 3, 4)
        val birthDate2 = LocalDate(1992, 7, 18)
        val student1 = Student(321, "Bill", birthDate1, emptyList(), 0.0)
        val subjects = listOf(Subject(SubjectName.COMPUTER_SCIENCE, 5.5f), Subject(SubjectName.MUSIC, 4.8f))
        val student2 = Student(123, "Alice", birthDate2, subjects, 5.15)
        return listOf(student1, student2)
    }
}
```

What we are doing here is creating a test server in the `configure()` function. This serves as our system under test, and we are mocking
out the student service class to always return the same students each time it is called. We then assert that the HTTP status code returned from the
API call is 200 (OK) and that the expected students are returned. Now run the tests and see that they fail because we have not implemented the code
in the controller to call the student service. Let's add that into the controller, it should look like this:

```kotlin
class StudentController(application: Application) : AbstractDIController(application) {
  private val studentService: StudentService by instance()

  override fun Route.getRoutes() {
    route("/students") {
      get {
        TODO()
      }

      get("{id}") {
        TODO()
      }

      post {
        TODO()
      }

      put("{id}") {
        TODO()
      }

      delete {
        TODO()
      }
    }
  }
}
```

## GET /students/{id}

In this part, we will be writing the test and the code for the endpoint that retrieves a single student by their id. We have the following requirements
that we need to meet for the endpoint.

- If the given student id does not match any in the system then it must return a 404 (Not Found) status code
- If the given student id is not an integer then it must return a 400 (Bad Request) status code
- Otherwise, if all the above conditions are not met then it must return a 200 (OK) status code and the matching student

Let's start by creating the test. Add the following code to your test file to test all these cases.

```kotlin
// setup code to add 
every { studentService.getStudentById(any()) } returns getTestStudent()
// test code to add 
@Test
fun `should return a single student when GET request is made to students endpoint with a valid id`() {
    withTestApplication(configure()) {
        handleRequest(HttpMethod.Get, "/students/123").apply {
            assertThat(response.status()).isEqualTo(HttpStatusCode.OK)
            val expectedStudent = getTestStudent()
            val rawResponse = response.content ?: ""
            val actualStudent = Json.decodeFromString<Student>(rawResponse)
            assertThat(actualStudent).isEqualTo(expectedStudent)
        }
    }
}

@Test
fun `should return a 404 status code when GET request is made to students endpoint with an id that does not match any students`() {
    every { studentService.getStudentById(any()) } returns null
    withTestApplication(configure()) {
        handleRequest(HttpMethod.Get, "/students/999").apply {
            assertThat(response.status()).isEqualTo(HttpStatusCode.NotFound)
        }
    }
}

@Test
fun `should return a 400 status code when GET request is made to students endpoint with an id that is not an integer`() {
    withTestApplication(configure()) {
        handleRequest(HttpMethod.Get, "/students/abc").apply {
            every { studentService.getStudentById(any()) } returns null
            assertThat(response.status()).isEqualTo(HttpStatusCode.BadRequest)
        }
    }
}
```

Notice when we add the test code we get a syntax error saying that there is a type mismatch on the mock of the student service. This is one of
the many ways that TDD helps to write better code. Sure you can quickly go and change the code to return the correct type, but when practicing TDD
you actually take a moment to pause and think about your design. In our case, we need to think about how will we represent the fact we did not find
a student with the given id. Will we throw some sort of exception? Will we return a null value? Or will we return some default value? I rarely
find myself doing this when I write the code first and the tests later. I think it's mainly because I am in a mindset where I need to write code as
fast as possible without putting too much thought into it. I then realise that I've created some big mess, and it has become too difficult to test
or, I am just too lazy to write the tests because of some poor design choices. I have been in many situations like that and that is why I have come
to like TDD more and more. It forces me to stop, and think about the code I am about to write instead of just picking the first thing that I think
will work.

Let's fix the syntax error by making the function in the student service return a nullable of type student. After you have done that go ahead and
run the tests again. This time everything should compile, but two of the tests should be failing.

Now that we have the tests ready, and we are happy with them, we can start writing the actual code without fear of making a mistake when refactoring
because we have the safety of the tests. Add the following code to your student by id endpoint in the controller, and it should make all the tests
pass.

```kotlin
get("{id}") {
    val id = call.parameters["id"]?.toIntOrNull()

    if (id == null) {
        call.respond(HttpStatusCode.BadRequest)
        return@get
    }

    studentService.getStudentById(id)?.let {
         call.respond(it)
         return@get
    }

    call.respond(HttpStatusCode.NotFound)
}
```

Make sure to run the tests and ensure they pass. If they do not go ahead and fix them up before you move on.

## POST /students

In this section, we will be implementing the endpoint that allows us to add a new student. We have the following requirements that we need to meet
for this endpoint.

- 
  If the given request does not match the API spec we should return a 400 (Bad Request) status code.
  We must also include a message explaining why the request is malformed
- If the given request matches the spec we should do the following:
  - Return a 201 (Created) status code
  - Return an id for the newly created student
  - Persist the new student, so they can be retrieved in subsequent requests such as getting all students or getting a student by their id

Note: In this example, we will be implementing the id incrementing in the student service and not the controller.
Do not worry about that for now, in the tests we will mock this out.

Add the following code to our `StudentControllerTestCase` class:
```kotlin
@Test
fun `should return a 201 status code when POST request is made to students endpoint that matches the expected api spec`() {
    every { studentService.addStudent(any()) } returns 123
    withTestApplication(configure()) {
        handleRequest(HttpMethod.Post, "/students") {
            addHeader("content-type", "application/json")
            setBody(getValidAddStudent())
        }.apply {
            assertThat(response.status()).isEqualTo(HttpStatusCode.Created)
        }
    }
}

@Test
fun `should return the new student id when POST request is made to students endpoint that matches the expected api spec`() {
    every { studentService.addStudent(any()) } returns 123
    withTestApplication(configure()) {
        handleRequest(HttpMethod.Post, "/students") {
            addHeader("content-type", "application/json")
            setBody(getValidAddStudent())
        }.apply {
            assertThat(response.status()).isEqualTo(HttpStatusCode.Created)
            val rawResponse = response.content ?: ""
            val addStudentResponse = Json.decodeFromString<AddStudentResponse>(rawResponse)
            assertThat(addStudentResponse.id).isEqualTo(123)
        }
    }
}

@Test
fun `should return a 400 status code when POST request is made to students endpoint that does not the expected api spec`() {
    every { studentService.addStudent(any()) } returns 123
    withTestApplication(configure()) {
        handleRequest(HttpMethod.Post, "/students") {
            addHeader("content-type", "application/json")
            setBody(getInvalidAddStudent())
        }.apply {
            assertThat(response.status()).isEqualTo(HttpStatusCode.BadRequest)
        }
    }
}

private fun getValidAddStudent(): String {
    return "{\n" +
            "    \"name\": \"Tom Banks\",\n" +
            "    \"dateOfBirth\": \"2021-11-04\",\n" +
            "    \"enrolledClasses\": [],\n" +
            "    \"averageGpa\": \"0.0\"\n" +
            "}"
}

private fun getInvalidAddStudent(): String {
    return "{\n" +
            "    \"name\": \"Tom Banks\",\n" +
            "    \"dateOfBirth\": \"2021-aa-04\",\n" +
            "    \"enrolledClasses\": [],\n" +
            "    \"averageGpa\": \"ccc\"\n" +
            "}"
}
```

Add the following code to the `StudentService` class:
```kotlin
fun addStudent(student: AddStudentRequest): Int {
    TODO()
}
```

Create a new model file called `AddStudent` with the following contents:
```kotlin
package nz.kirillov.model

import kotlinx.datetime.LocalDate
import kotlinx.datetime.serializers.LocalDateIso8601Serializer
import kotlinx.serialization.Serializable

@Serializable
data class AddStudentRequest(
    val name: String,
    @Serializable(with = LocalDateIso8601Serializer::class)
    val dateOfBirth: LocalDate,
    val enrolledClasses: List<Subject>,
    val averageGpa: Double
)

@Serializable
data class AddStudentResponse(val id: Int)
```

I have decided to generate the IDs for students on the server so a separate DTO class is needed for the request to add a student.
As always let's start by running the tests and see that the new ones we have added have failed. Now we can write the code to make the tests pass.
Add the following code and re-run the tests:

```kotlin
post {
    val student: AddStudentRequest
    try {
        student = call.receive()
    } catch (e: Exception) {
        println(e)
        call.respond(HttpStatusCode.BadRequest, e.message ?: "Bad Request")
        return@post
    }

    val newId = studentService.addStudent(student)
    call.respond(HttpStatusCode.Created, AddStudentResponse(newId))
}
```

They should now all be passing!

## PUT /students/{id} and DELETE /students/{id}

I will not be showing the code for these two endpoints in the tutorial as it will make this tutorial far too long. You can still find the rest of
the code in the GitHub repo. For your reference, these are the requirements for the two endpoints.

### Update Student

- Return a 200 (OK) status code if the student is updated successfully
- When updating a student the average GPA must be recalculated and saved
- Return a 400 (Bad Request) status code if an update to a field is attempted that does not match the API spec

### Delete Student

- Return a 200 (OK) status code if the student was deleted successfully
- Return a 400 (Bad Request) status code if the student id is not an integer
- Return a 404 (Not Found) status code if the student to be deleted with the given id does not exist

## TDD for the Service

Now that we have the controller working, at least a few of the endpoints, we can test our API a bit more in-depth, and we can start fleshing out the
student service class. This will be where the core of our business logic will reside. To get started create a new class in the tests
directory called `StudentServiceTestCase`

This is where we will use separation of concerns to make developing and testing our code much easier. Instead of putting all the logic into the
controller, we will separate out the validation logic and core business logic. The controller will do some simple validation like making sure all the
fields are correct and match the API schema (most frameworks will do this for you already). We will then implement the requirements for the
endpoint in the service classes. For example, when updating a student we will recalculate the average GPA in the service class instead of the
controller to separate things out and make it easier to test and extend in the future.

Add the following tests to the service class, and then we can get to writing the code to satisfy the test cases.

```kotlin
class StudentServiceTestCase {

    @MockK
    private lateinit var studentRepository: StudentRepository

    private lateinit var service: StudentService

    @Before
    fun setUp() {
        MockKAnnotations.init(this)
        service = StudentService(studentRepository)
    }

    @Test
    fun `should return all students when getting all students and there are students`() {
        val expectedStudents = getTestStudents()
        every { studentRepository.getStudents() } returns getTestStudents()

        val result = service.getStudents()
        assertThat(result).isEqualTo(expectedStudents)
        verify { studentRepository.getStudents() }
    }

    @Test
    fun `should return empty collection of students when getting all students and there are no students`() {
        every { studentRepository.getStudents() } returns emptyList()

        val result = service.getStudents()

        assertThat(result).isEmpty()
        verify { studentRepository.getStudents() }
    }

    @Test
    fun `should call repository when getting a single student`() {
        val expectedStudent = getTestStudent()
        every { studentRepository.getStudentById(any()) } returns expectedStudent

        val result = service.getStudentById(123)

        assertThat(result).isEqualTo(expectedStudent)
        verify { studentRepository.getStudentById(123) }
    }

    @Test
    fun `should return null when getting a single student that does not exist`() {
        every { studentRepository.getStudentById(any()) } returns null

        val result = service.getStudentById(123)

        assertThat(result).isNull()
        verify { studentRepository.getStudentById(123) }
    }

    @Test
    fun `should return the next highest id for a student when adding a new student`() {
        val testStudent = getTestStudent()
        val addStudentRequest = AddStudentRequest(testStudent.name, testStudent.dateOfBirth, testStudent.enrolledClasses, testStudent.averageGpa)
        val nextStudentId = 322
        every { studentRepository.addStudent(any()) } returns nextStudentId

        val result = service.addStudent(addStudentRequest)

        assertThat(result).isEqualTo(nextStudentId)
        verify { studentRepository.addStudent(addStudentRequest) }
    }

    @Test
    fun `should call repository when updating a student`() {
        val studentId = 123
        val updateStudentRequest = getUpdatedStudent(studentId).first
        val updatedStudent = getUpdatedStudent(studentId).second
        every { studentRepository.getStudentById(123) } returns getTestStudent()
        every { studentRepository.updateStudent(any()) } returns updatedStudent

        service.updateStudent(studentId, updateStudentRequest)

        verify { studentRepository.updateStudent(updatedStudent) }
    }

    @Test
    fun `should update student fields correctly when updating a student`() {
        val studentId = 123
        val updateStudentRequest = getUpdatedStudent(studentId).first
        val updatedStudent = getUpdatedStudent(studentId).second
        every { studentRepository.getStudentById(123) } returns getTestStudent()
        every { studentRepository.updateStudent(any()) } returns updatedStudent

        val result = service.updateStudent(studentId, updateStudentRequest)

        assertThat(result.name).isEqualTo("Timothy")
        assertThat(result.dateOfBirth).isEqualTo(LocalDate(2001, 12, 12))
        assertThat(result.enrolledClasses).isEqualTo(UPDATED_SUBJECTS)
        assertThat(result.averageGpa).isEqualTo(3.0)
    }

    @Test
    fun `should re-calculate student GPA when student is updated`() {
        val studentId = 123
        val updateStudentRequest = getUpdatedStudent(studentId).first
        val updatedStudent = getUpdatedStudent(studentId).second
        every { studentRepository.getStudentById(123) } returns getTestStudent()
        every { studentRepository.updateStudent(any()) } returns updatedStudent

        val result = service.updateStudent(studentId, updateStudentRequest)

        assertThat(result.averageGpa).isEqualTo(3.0)
    }

    @Test
    fun `should throw error when trying to update a student that does not exist`() {
        val studentId = 123
        val updateStudentRequest = getUpdatedStudent(studentId).first
        every { studentRepository.getStudentById(studentId) } returns null

        assertThrows(StudentDoesNotExistException::class.java) {
            service.updateStudent(studentId, updateStudentRequest)
        }
    }

    @Test
    fun `should call repository when deleting a student`() {
        val studentId = 123
        every { studentRepository.deleteStudent(any()) } returns Unit

        service.deleteStudent(studentId)

        verify { studentRepository.deleteStudent(studentId) }
    }

    @Test
    fun `should throw error if student does not exist when trying to delete student`() {
        val studentId = 123
        every { studentRepository.deleteStudent(any()) } throws StudentDoesNotExistException()

        assertThrows(StudentDoesNotExistException::class.java) {
            service.deleteStudent(studentId)
        }
    }

    private fun getTestStudents(): List<Student> {
        val student1 = getTestStudent()
        val birthDate2 = LocalDate(1992, 7, 18)

        val subjects = listOf(Subject(SubjectName.COMPUTER_SCIENCE, 5.5f), Subject(SubjectName.MUSIC, 4.8f))
        val student2 = Student(321, "Alice", birthDate2, subjects, 5.15)
        return listOf(student1, student2)
    }

    private fun getUpdatedStudent(id: Int): Pair<UpdateStudentRequest, Student> {
        val name = "Timothy"
        val updatedBirthDate = LocalDate(2001, 12, 12)
        val subjects = UPDATED_SUBJECTS
        val averageGpa = subjects.map{it.grade}.average()

        val student = Student(id, name, updatedBirthDate, subjects, averageGpa)
        return Pair(UpdateStudentRequest(name, updatedBirthDate, subjects), student)
    }

    private fun getTestStudent(): Student {
        val birthDate1 = LocalDate(1990, 3, 4)
        return Student(123, "Bill", birthDate1, emptyList(), 0.0)
    }

    companion object {
        val UPDATED_SUBJECTS = listOf(
            Subject(SubjectName.BIOLOGY, 2.0f),
            Subject(SubjectName.ASTRONOMY, 4.0f),
            Subject(SubjectName.PHYSICS, 3.0f)
        )
    }
}
```

Whoa! That is quite a lot of code, but those ~190 lines of code should be enough to give us confidence that our application will work.
At this point, it would be good to submit a pull request for your colleagues to take a look at what you've come up with.
This gives you many benefits that you wouldn't normally get if you had written the code upfront and the tests afterward.
Perhaps you misinterpreted some requirements, or maybe there is a flaw in the design of the code that they can now see that they may have overlooked
previously. Another reason why getting a review for only tests is that the test code will actually get scrutinized properly. I think most of the time
developers (I am guilty of this myself 😅) skip through the tests in the PR because they have already spent a lot of time peering
over the code, especially if it's a large PR. Most of the time test code is only properly reviewed when the next developer comes along to fix or
update tests and that is usually a frustrating time.

## Student Service Code

Now that we have our tests written we can start working on the actual code. Even when writing the code myself I had to make small adjustments to
the tests and how the code is structured. This is completely normal because chances are you are never going to write something 100%
correct the first time around. This is why you should make small incremental changes instead of doing a week's worth of work only to realise you
had an incorrect assumption, realise the requirements were not clear and were ambiguous.
Add the following code to satisfy the tests:

```kotlin
class StudentService(private val repository: StudentRepository) {

    fun getStudents(): List<Student> {
        return repository.getStudents()
    }

    fun getStudentById(id: Int): Student? {
        return repository.getStudentById(id)
    }

    fun addStudent(student: AddStudentRequest): Int {
        return repository.addStudent(student)
    }

    fun updateStudent(studentId: Int, updatedStudent: UpdateStudentRequest): Student {
        repository.getStudentById(studentId) ?: throw StudentDoesNotExistException("Student with id: $studentId does not exist")

        val averageGpa = updatedStudent.enrolledClasses.map{ it.grade }.average()
        val student = Student(studentId, updatedStudent.name, updatedStudent.dateOfBirth, updatedStudent.enrolledClasses, averageGpa)
        return repository.updateStudent(student)
    }

    fun deleteStudent(id: Int) {
        repository.deleteStudent(id)
    }
}
```

This is obviously a simplified example, in the real world, your business logic will most likely be far more complex than this so writing the tests
before the code is even more important. One other reason is that writing the tests before the code makes you actually go and read the code properly,
get an understanding of how it works and how it fits into the system. It will seem like a waste of time going through all the code and trying to wrap
your head around it, but this will help you understand the system as a whole better, and you can make better architectural decisions later down the
track. You might be able to see areas that can be refactored to make the code simpler and more re-usable, or you could see a way to make the code
more performant.

## Student Repository Tests

We are almost there! Just some more tests to write, and we will be good to go. Start off by creating a new class called:
`StudentRepositoryTestCase`. Inside it add the following code to satisfy our requirements:

```kotlin
class StudentRepositoryTestCase {

    private lateinit var repository: StudentRepository

    @Before
    fun setUp() {
        resetTestFile()
        repository = StudentRepository()
    }

    @Test
    fun `should return all students when getting all students and there are students`() {
        val expectedStudents = getTestStudents()
        writeDataToFile(expectedStudents)

        val actualStudents = repository.getStudents()
        assertThat(actualStudents).isEqualTo(expectedStudents)
    }

    @Test
    fun `should return no students when getting all students and there are no students`() {
        val actualStudents = repository.getStudents()

        assertThat(actualStudents).isEmpty()
    }

    @Test
    fun `should return correct student when getting a student by id`() {
        val expectedStudent = getTestStudent()
        writeDataToFile(listOf(expectedStudent))

        val actualStudent = repository.getStudentById(expectedStudent.id)
        assertThat(actualStudent).isNotNull
        assertThat(actualStudent).isEqualTo(expectedStudent)
    }

    @Test
    fun `should return null when getting a student by id that does not exist`() {
        writeDataToFile(listOf(getTestStudent()))

        val actualStudent = repository.getStudentById(NON_EXISTENT_STUDENT_ID)
        assertThat(actualStudent).isNull()
    }

    @Test
    fun `should return next highest id when adding a student`() {
        val students = getTestStudents()
        writeDataToFile(students)
        val lastId = students.last().id
        val expectedNewId = lastId + 1

        val studentToAdd = getAddStudentRequest()

        val actualNewId = repository.addStudent(studentToAdd)
        assertThat(actualNewId).isEqualTo(expectedNewId)
    }

    @Test
    fun `should update student when student exists`() {
        val students = getTestStudents()
        writeDataToFile(students)
        val oldStudent = students.last()

        val newName = "Phillip Morrison"
        val newDateOfBirth = LocalDate(1998, 5, 5)
        val studentToUpdateTo = Student(oldStudent.id, newName, newDateOfBirth, oldStudent.enrolledClasses, oldStudent.averageGpa)

        val updatedStudent = repository.updateStudent(studentToUpdateTo)
        assertThat(updatedStudent).isEqualTo(studentToUpdateTo)

        val updatedStudentFromRepository = repository.getStudentById(oldStudent.id)
        assertThat(updatedStudentFromRepository).isNotNull
        assertThat(updatedStudentFromRepository).isEqualTo(updatedStudent)
    }

    @Test
    fun `should throw error when trying to update student that does not exist`() {
        val newName = "Phillip Morrison"
        val newDateOfBirth = LocalDate(1998, 5, 5)
        val studentToUpdateTo = Student(NON_EXISTENT_STUDENT_ID, newName, newDateOfBirth, emptyList(), 0.0)

        assertThrows(StudentDoesNotExistException::class.java) {
            repository.updateStudent(studentToUpdateTo)
        }
    }

    @Test
    fun `should delete student when given student exists`() {
        val students = getTestStudents()
        writeDataToFile(students)
        val studentToDelete = students.last()

        repository.deleteStudent(studentToDelete.id)
        val actualStudent = repository.getStudentById(studentToDelete.id)
        assertThat(actualStudent).isNull()
    }

    @Test
    fun `should throw error when trying to delete student that does not exist`() {
        assertThrows(StudentDoesNotExistException::class.java) {
            repository.deleteStudent(NON_EXISTENT_STUDENT_ID)
        }
    }

    private fun resetTestFile() {
        val writer = getStudentFileWriter()
        writer.write("[]")
        writer.close()
    }

    private fun getStudentFileWriter(): FileWriter {
        return FileWriter(this::class.java.classLoader.getResource("students.json")!!.file)
    }

    private fun writeDataToFile(students: List<Student>) {
        val writer = getStudentFileWriter()
        val data = Json.encodeToString(students)

        writer.write(data)
        writer.close()
    }

    private fun getAddStudentRequest(): AddStudentRequest {
        return AddStudentRequest("Bilbo Baggins", LocalDate(1892, 1, 3), listOf(Subject(SubjectName.GEOGRAPHY,  9.0f)), 9.0)
    }

    companion object {
        const val NON_EXISTENT_STUDENT_ID = 999999
    }
}
```

In this example, we are using a local JSON file as our data store, but this concept can be applied to any other form of data storage
(relational database, NoSQL database, etc.). They are all built around similar concepts, they abstract away the implementation and provide an API
for us to consume. We do not need to understand how exactly Postgres, Mongo, Firebase, or any other form of data store implements their logic to
read and save data. We just rely on the contract that is provided with the API, and we are able to mock out the implementation of the data store in
our tests so that we are sure our code is working without worrying about testing the data store code itself. That code should be tested by the
authors of the data store.

## Student Repository Code

This is the code that will satisfy all the tests for the student repository. It might not be the most performant or most elegant solution, but
now that we have tests in place we can refactor to our heart's content without worrying that we break the implementation. Once all tests are green
then we should have enough confidence in starting the next step in the development process. For example, we could push our code and have a peer
review, or perhaps we pair programmed this bit of code, and we are ready to release it to production.

## Conclusion

In this post, we have covered a number of different topics, strategies, and principles to apply when developing code in a TDD fashion. We have used
separation of concerns by creating clear and well-encapsulated boundaries between different areas of the system to split up the code into logical
chunks which makes understanding the system much easier. We have also used mocking in conjunction with this to aid testing our code by swapping
out the behaviour of dependencies to test different code paths and scenarios. Finally, we used dependency injection to allow easy
substitution of dependencies when writing tests and to allow us to easily change the behaviour of those dependencies.

Hopefully, you have learned something useful from this blog and that TDD can be applied to 'real-life' scenarios too, it just takes a bit of time
to get your head around it. As a final exercise, I encourage you to extend this API and use a different data store (Redis, Postgres, Couchbase, etc.)
to write and persist the data for this student management system.
