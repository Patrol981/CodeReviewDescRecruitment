# Orginalny Kod
```c#
[HttpPost("delete/{id}")]
public void Delete(uint id)
{
	User user = _context.Users.FristOrDefault(user => user.Id == id);
	_context.Users.Remove(user);
	_context.SaveChanges();
	Debug.WriteLine($"The user with Login={user.login} has been deleted.");
	return Ok();
}
```

Zakladając że składniowo jest poprawny i się kompiluje, zwróciłbym uwagę na następujące elementy:
- ```User user``` zamieniłbym na ```var user```, nie ma potrzeby zaznaczania typu zmiennej jeżeli jej typ można wywnioskować z metody która ją zwraca
- Brak abstraktu <b>Repozytorium</b> i <b>Serwisu</b>, standardem jest organizacja kodu w taki sposób, żeby ten był czytelny i modularny. Rownież przy takiej implementacji <b>DDD</b> nie jest możliwe
- Osobiście bym nie zwracał <b>Ok</b> tylko <b>NoContent</b>
- w pierwszej linijce metody zamieniłbym wyrażenie lambda na ```x => x.Id == id```, ze względu na to że user został wcześniej zadeklarowany i jeżeli nie stanowi to problemu dla samego .NETa to wygląda bardzo myląco, bo tak na prawdę to lambda nie wchodzi w żadną interakcję z wcześniej zdefiniowanym userem
-  Nie stosowałbym ```Debug.WriteLine``` bezpośrednio, są od tego biblioteki do loggingu
- Zmieniłbym nazwę metody, żeby była czytelniejsza
- Owrappowałbym parametry w osobne klasy
- <b>POST</b> request nie jest tutaj potrzebny, wystarczy <b>DELETE</b>
- delete endpoint jest niepotrzebny, zastąpiłbym nazwą lokalną dla operacji
- Dodatkowo też poleciłbym spróbowanie <b>MinimalAPI</b>

# Przerobiony kod nieinwazyjny
```c#
[HttpDelete("users/{id}")]
public void DeleteUser(DeleteUserRequest deleteUserRequest)
{
	var user = _context.Users.FristOrDefault(x => x.Id == deleteUserRequest.Id);
	_context.Users.Remove(user);
	_context.SaveChanges();
	_logger.Log($"The user with Login={user.login} has been deleted.");
	return NoContent();
}
```

# Przerobiony kod inwazyjny (wariant modularny)
```c#
[HttpDelete("users/{id}")]
public void DeleteUser(DeleteUserRequest deleteUserRequest)
{
	var user = _userService.GetUser(deleteUserRequest.Id);
	_userService.DeleteUser(user);
	// Logger przeniesiony do UserService
	return NoContent();
}
```

# Przerobiony kod inwazyjny (wariant natychmiastowy)
```c#
[HttpDelete("users/{id}")]
public void DeleteUser(DeleteUserRequest deleteUserRequest)
{
	_userService.DeleteUser(deleteUserRequest.Id);
	// Logger przeniesiony do UserService
	return NoContent();
}
```

# Przerobiony kod inwazyjny (wariant modularny, Minimal API)
```c#
public void DefineEndpoints(WebApplication app)
{
	...
	app.MapDelete("users/{id}", DeleteUser);
	...
}

public static async Task<Results<NoContent>> DeleteUser(
	IUserService userService,
	DeleteUserRequest deleteUserRequest
)
{
	var user = _userService.GetUser(deleteUserRequest.Id);
	_userService.DeleteUser(user);
	return TypedResults.NoContent();
}
```

# Przerobiony kod inwazyjny (wariant natychmiastowy, Minimal API)
```c#
public void DefineEndpoints(WebApplication app)
{
	...
	app.MapDelete("users/{id}", DeleteUser);
	...
}

public static async Task<Results<NoContent>> DeleteUser(
	IUserService userService,
	DeleteUserRequest deleteUserRequest
)
{
	var user = _userService.DeleteUser(deleteUserRequest.Id);
	return TypedResults.NoContent();
}
```
