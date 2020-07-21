<a name="ddav"></a>
### <a name="disable-default-account-verification"></a><span data-ttu-id="ae670-101">Отключить проверку учетной записи по умолчанию</span><span class="sxs-lookup"><span data-stu-id="ae670-101">Disable default account verification</span></span>

<span data-ttu-id="ae670-102">При использовании шаблонов по умолчанию пользователь перенаправляется к, `Account.RegisterConfirmation` где можно выбрать ссылку для подтверждения учетной записи.</span><span class="sxs-lookup"><span data-stu-id="ae670-102">With the default templates, the user is redirected to the `Account.RegisterConfirmation` where they can select a link to have the account confirmed.</span></span> <span data-ttu-id="ae670-103">Значение по умолчанию `Account.RegisterConfirmation` используется ***только*** для тестирования, автоматическая проверка учетной записи должна быть отключена в рабочем приложении.</span><span class="sxs-lookup"><span data-stu-id="ae670-103">The default `Account.RegisterConfirmation` is used ***only*** for testing, automatic account verification should be disabled in a production app.</span></span>

<span data-ttu-id="ae670-104">Чтобы запросить подтвержденную учетную запись и предотвратить немедленный вход при регистрации, установите `DisplayConfirmAccountLink = false` в */Ареас/идентити/Пажес/аккаунт/регистерконфирматион.кштмл.КС*:</span><span class="sxs-lookup"><span data-stu-id="ae670-104">To require a confirmed account and prevent immediate login at registration, set `DisplayConfirmAccountLink = false` in */Areas/Identity/Pages/Account/RegisterConfirmation.cshtml.cs*:</span></span>

[!code-csharp[](~/security/authentication/identity/sample/WebApp3/Areas/Identity/Pages/Account/RegisterConfirmation.cshtml.cs?name=snippet&highlight=34)]