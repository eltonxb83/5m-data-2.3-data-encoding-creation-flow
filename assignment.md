# Assignment

## Brief

Write the Python codes for the following questions.

## Instructions

Paste the answer as Python in the answer code section below each question.

### Question 1

Question: Implement a simple Thrift server and client that defines a `Student` struct with fields `name` (string), `age` (integer), and `courses` (list of strings). Include a service `School` with a method `enrollCourse` that takes a `Student` record and a course name, adds the course to the student's course list, and returns the updated `Student` record.

Answer:

```python

# Thrift schema (student.thrift)

%%writefile ../schema/student.thrift

struct Student{
	1: required string name,
	2: optional i32 age,
	3: optional list<string> courses
}

service School{
	Student enrollCourse(1: Student student, 2: string course) 
}
 
# Thrift server (student_server.py)

%%writefile ../student_thrift_server.py
import thriftpy2

student_thrift = thriftpy2.load("../schema/student.thrift",module_name="student_thrift")

from thriftpy2.rpc import make_server

class School(object):
	def enrollCourse(self, student, course):
	  	student.courses.append(course)
		return student

server = make_server(student_thrift.School, School(), client_timeout=None)
server.serve() 

# Thrift client (student_client.py)

import thriftpy2
from thriftpy2.rpc import make_client 

student_thrift = thriftpy2.load("../schema/student.thrift", module_name = "student_thrift")

school = make_client(student_thrift.School, timeout=None) 


```

### Question 2

Question: Implement a simple Protocol Buffers server and client that defines a `Book` message with fields `title` (string), `author` (string), and `page_count` (integer). Include a service `Library` with a method `checkoutBook` that takes a `Book` message and returns the same `Book` message.

Answer:

```python

# Protobuf schema (book.proto)

%%writefile ../schema/book.proto
syntax = "proto3"

message Book{
	string title =1;
	string author =2;
	int32 page_count =3;
}

service Library{
	rpc checkoutBook(Book) returns (Book) {}
}

# Protobuf server (book_server.py)

%%writefile ../book_protobuf_server.py
from concurrent import futures
import grpc  
import book_pb2 
import book_pb2_grpc

class Library(book_pb2_grpc.LibraryServicer):
	def checkoutBook(self, request, context):
		return request

server = grpc.server(futures.ThreadPoolExecutor(max_workers=2))
book_pb2_grpc.add_LibraryServicer_to_server(
	Library(), server)
server.add_insecure_port('[::]:50051')
server.start()
server.wait_for_termination()  

# Protobuf client (book_client.py)

import sys
sys.path.append('..')
import grpc
import book_pb2
import book_pb2_grpc

channel = grpc.insecure_channel("localhost:50051")
stub = book_pb2_grpc.LibraryStub(channel)

book = book_pb2.Book(
    title=title,
    author=author,
    page_count=page_count
)

response = stub.checkoutBook(book)

print("Title:",response.title)
print("Author:", response.author)
print("Pages:", response.page_count)       

```

## Submission

- Submit the URL of the GitHub Repository that contains your work to NTU black board.
- Should you reference the work of your classmate(s) or online resources, give them credit by adding either the name of your classmate or URL.
