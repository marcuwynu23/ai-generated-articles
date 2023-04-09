# Implementing CRUD RESTful API in ASP.NET

Implementing a CRUD (Create, Read, Update, Delete) RESTful API is a common task when building web applications. ASP.NET, a popular web development framework, makes it easy to create a RESTful API that can handle CRUD operations on a data model. In this article, we'll walk through the steps to create a simple CRUD RESTful API in ASP.NET using a sample Book model. We'll create a BooksController that handles GET, POST, PUT, and DELETE requests for books, and we'll use Postman to test the API. By the end of this article, you'll have a basic understanding of how to implement a CRUD RESTful API in ASP.NET.

## Creating a Model

First, create a model that represents the data you want to store. For example, if you want to store data about books, you could create a Book model:

```csharp
public class Book
{
    public int Id { get; set; }
    public string Title { get; set; }
    public string Author { get; set; }
    public int YearPublished { get; set; }
}
```

## Creating a Controller

Next, create a controller that handles CRUD operations on the Book model. For example, you could create a BooksController with the following methods:

```csharp
[Route("api/[controller]")]
[ApiController]
public class BooksController : ControllerBase
{
    private readonly List<Book> _books = new List<Book>();

    [HttpGet]
    public IEnumerable<Book> Get()
    {
        return _books;
    }

    [HttpGet("{id}")]
    public Book Get(int id)
    {
        return _books.FirstOrDefault(b => b.Id == id);
    }

    [HttpPost]
    public void Post([FromBody] Book book)
    {
        _books.Add(book);
    }

    [HttpPut("{id}")]
    public void Put(int id, [FromBody] Book book)
    {
        var existingBook = _books.FirstOrDefault(b => b.Id == id);
        if (existingBook != null)
        {
            existingBook.Title = book.Title;
            existingBook.Author = book.Author;
            existingBook.YearPublished = book.YearPublished;
        }
    }

    [HttpDelete("{id}")]
    public void Delete(int id)
    {
        var existingBook = _books.FirstOrDefault(b => b.Id == id);
        if (existingBook != null)
        {
            _books.Remove(existingBook);
        }
    }
}
```

In this example, the Get method returns a list of all books, the Get(int id) method returns a single book by ID, the Post method adds a new book, the Put(int id) method updates an existing book, and the Delete(int id) method deletes a book by ID.

## Testing the API

To test the API, you can use a tool like Postman to send HTTP requests to the API endpoints. For example, to get a list of all books, you could send a GET request to http://localhost:5000/api/books. To add a new book, you could send a POST request with the book data in the request body.