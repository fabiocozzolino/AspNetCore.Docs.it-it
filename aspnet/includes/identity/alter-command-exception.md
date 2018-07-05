> <span data-ttu-id="899e4-101">Alcuni comandi non sono supportati se l'app Usa SQLite come archivio dati di identità.</span><span class="sxs-lookup"><span data-stu-id="899e4-101">Some commands aren't supported if the app uses SQLite as its Identity data store.</span></span> <span data-ttu-id="899e4-102">A causa delle limitazioni nel motore di database, `Alter` comandi generano l'eccezione seguente:</span><span class="sxs-lookup"><span data-stu-id="899e4-102">Due to limitations in the database engine, `Alter` commands throw the following exception:</span></span>
>
> <span data-ttu-id="899e4-103">"System. NotSupportedException: SQLite non supporta questa operazione di migrazione."</span><span class="sxs-lookup"><span data-stu-id="899e4-103">"System.NotSupportedException: SQLite does not support this migration operation."</span></span> 
>
> <span data-ttu-id="899e4-104">Come per risolvere il problema, eseguire le migrazioni Code First per modificare le tabelle nel database.</span><span class="sxs-lookup"><span data-stu-id="899e4-104">As a work around, run Code First migrations on the database to change the tables.</span></span>