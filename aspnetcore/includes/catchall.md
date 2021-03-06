> [!WARNING]
> Un parametro **catch-all** può corrispondere erroneamente alle route a causa di un [bug](https://github.com/dotnet/aspnetcore/issues/18677) nel routing. Le app interessate da questo bug hanno le seguenti caratteristiche:
>
> * Una route catch-all, ad esempio`{**slug}"`
> * La route catch-all non riesce a trovare le richieste corrispondenti.
> * La rimozione di altre route rende l'avvio del funzionamento della route catch-all.
>
> Vedere i bug di GitHub [18677](https://github.com/dotnet/aspnetcore/issues/18677) e [16579](https://github.com/dotnet/aspnetcore/issues/16579) per i casi di esempio che hanno raggiunto questo bug.
>
> È prevista una correzione del consenso esplicito per questo bug. Questo documento verrà aggiornato al rilascio della patch. Quando la patch viene rilasciata, il codice seguente imposta un commutatore interno che corregge il bug:
>
>```
>public static void Main(string[] args)
>{
>    AppContext.SetSwitch("Microsoft.AspNetCore.Routing.UseCorrectCatchAllBehavior", true);
>    CreateHostBuilder(args).Build().Run();
>}
>// Remaining code removed for brevity.
>```