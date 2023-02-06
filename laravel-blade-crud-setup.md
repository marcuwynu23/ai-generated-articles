# Laravel Blade View CRUD SETUP

- ### Listing Data
  retrieve all Todo items from the database
  return the todo.index view and pass the todos variable to it

```php
public function index()
{

	$todos = Todo::all();
  return view("todo.index", compact('todos'));

}
```

- ### Create Data
  return the todo.create view

```php
public function create()
{
	return view('todo.create');
}
```

- ### Storing Data
  create a new Todo item using the data from the request object
  redirect the user back to the index page

```php
public function store(Request $request)
{
	Todo::create($request->all());
	return redirect()->route('todo.index');

}
```

```html
<form method="post" action="{{ route('todo.store') }}">
  @csrf @method('POST')
  <div class="form-group">
    <label for="title">Title</label>
    <input type="text" class="form-control" id="title" name="title" />
  </div>
  <div class="form-group">
    <label for="content">Content</label>
    <textarea
      class="form-control"
      id="content"
      name="content"
      rows="3"
    ></textarea>
  </div>
  <div class="form-group">
    <label for="status">Status</label>
    <select class="form-control" id="status" name="status">
      <option value="todo">To do</option>
      <option value="inprogress">In progress</option>
      <option value="done">Done</option>
    </select>
  </div>
  <button type="submit" class="btn btn-primary">Submit</button>
</form>
```

- ### Showing Data by id
  redirect the user back to the index page

```php
public function show($id)
{
	return redirect()->route('todo.index');
}
```

- ### Editing Data and Show it in form
  return the todo.edit view and pass the todo variable to it

```php
public function edit(Todo $todo)
{
	return view('todo.edit', compact("todo"));
}
```

```html
<a
  class="btn btn-primary btn-sm d-inline-block"
  href="{{ route('todo.edit', $todo) }}"
  >Edit</a
>
```

- ### Updating Data from Edit Form
  update the Todo item using the data from the request object
  redirect the user back to the index page

```php
public function update(Request $request, Todo $todo)
{
	$todo->update($request->all());
	return redirect()->route('todo.index');
}
```

```html
<form method="post" action="{{ route('todo.update', $todo) }}">
  @csrf @method('PATCH')
  <div class="form-group">
    <label for="title">Title</label>
    <input
      type="text"
      class="form-control"
      id="title"
      name="title"
      value="{{ old('title', $todo->title) }}"
    />
  </div>
  <div class="form-group">
    <label for="content">Content</label>
    <textarea class="form-control" id="content" name="content" rows="3">
{{ old('content', $todo->content) }}</textarea
    >
  </div>
  <div class="form-group">
    <label for="status">Status</label>
    <select
      class="form-control"
      id="status"
      name="status"
      default="{{ old('status', $todo->status) }}"
    >
      <option value="todo">To do</option>
      <option value="inprogress">In progress</option>
      <option value="done">Done</option>
    </select>
  </div>
  <button type="submit" class="btn btn-primary">Submit</button>
</form>
```

- ### Delete Data from Table
  delete the Todo item
  redirect the user back to the index page

```php
public function destroy(Todo $todo)
{
	$todo->delete();
  return redirect()->route('todo.index');
}
```

```html
<form method="post" action="{{ route('todo.destroy', $todo) }}">
  @csrf @method('DELETE')
  <button class="btn btn-primary btn-sm d-inline-block">Delete</button>
</form>
```
