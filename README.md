[![FIWARE Banner](https://fiware.github.io/tutorials.Roles-Permissions/img/fiware.png)](https://www.fiware.org/developers)

[![FIWARE Security](https://img.shields.io/badge/FIWARE-Security-ff7059.svg?logo=data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABsAAAAVCAYAAAC33pUlAAAABHNCSVQICAgIfAhkiAAAA8NJREFUSEuVlUtIFlEUx+eO+j3Uz8wSLLJ3pBiBUljRu1WLCAKXbXpQEUFERSQF0aKVFAUVrSJalNXGgmphFEhQiZEIPQwKLbEUK7VvZrRvbr8zzjfNl4/swplz7rn/8z/33HtmRhn/MWzbXmloHVeG0a+VSmAXorXS+oehVD9+0zDN9mgk8n0sWtYnHo5tT9daH4BsM+THQC8naK02jCZ83/HlKaVSzBey1sm8BP9nnUpdjOfl/Qyzj5ust6cnO5FItJLoJqB6yJ4QuNcjVOohegpihshS4F6S7DTVVlNtFFxzNBa7kcaEwUGcbVnH8xOJD67WG9n1NILuKtOsQG9FngOc+lciic1iQ8uQGhJ1kVAKKXUs60RoQ5km93IfaREvuoFj7PZsy9rGXE9G/NhBsDOJ63Acp1J82eFU7OIVO1OxWGwpSU5hb0GqfMydMHYSdiMVnncNY5Vy3VbwRUEydvEaRxmAOSSqJMlJISTxS9YWTYLcg3B253xsPkc5lXk3XLlwrPLuDPKDqDIutzYaj3eweMkPeCCahO3+fEIF8SfLtg/5oI3Mh0ylKM4YRBaYzuBgPuRnBYD3mmhA1X5Aka8NKl4nNz7BaKTzSgsLCzWbvyo4eK9r15WwLKRAmmCXXDoA1kaG2F4jWFbgkxUnlcrB/xj5iHxFPiBN4JekY4nZ6ccOiQ87hgwhe+TOdogT1nfpgEDTvYAucIwHxBfNyhpGrR+F8x00WD33VCNTOr/Wd+9C51Ben7S0ZJUq3qZJ2OkZz+cL87ZfWuePlwRcHZjeUMxFwTrJZAJfSvyWZc1VgORTY8rBcubetdiOk+CO+jPOcCRTF+oZ0okUIyuQeSNL/lPrulg8flhmJHmE2gBpE9xrJNkwpN4rQIIyujGoELCQz8ggG38iGzjKkXufJ2Klun1iu65bnJub2yut3xbEK3UvsDEInCmvA6YjMeE1bCn8F9JBe1eAnS2JksmkIlEDfi8R46kkEkMWdqOv+AvS9rcp2bvk8OAESvgox7h4aWNMLd32jSMLvuwDAwORSE7Oe3ZRKrFwvYGrPOBJ2nZ20Op/mqKNzgraOTPt6Bnx5citUINIczX/jUw3xGL2+ia8KAvsvp0ePoL5hXkXO5YvQYSFAiqcJX8E/gyX8QUvv8eh9XUq3h7mE9tLJoNKqnhHXmCO+dtJ4ybSkH1jc9XRaHTMz1tATBe2UEkeAdKu/zWIkUbZxD+veLxEQhhUFmbnvOezsJrk+zmqMo6vIL2OXzPvQ8v7dgtpoQnkF/LP8Ruu9zXdJHg4igAAAABJRU5ErkJgggA=)](https://www.fiware.org/developers/catalogue/)
[![Documentation](https://readthedocs.org/projects/fiware-tutorials/badge/?version=latest)](https://fiware-tutorials.readthedocs.io/en/latest)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)







# Start Up

To start the installation, do the following:

```console
git clone git@github.com:Fiware/tutorials.Roles-Permissions.git
cd tutorials.Roles-Permissions

./services create
```

>**Note** The initial creation of Docker images can take up to three minutes


Thereafter, all services can be initialized from the command line by running the [services](https://github.com/Fiware/tutorials.Roles-Permissions/blob/master/services) Bash script provided within the repository:

```console
./services <command>
```

Where `<command>` will vary depending upon the exercise we wish to activate.

>:information_source: **Note:** If you want to clean up and start over again you can do so with the following command:
>
>```console
>./services stop
>```
>


### Dramatis Personae

The following people at `test.com` legitimately have accounts within the Application

* Alice, she will be the Administrator of the **Keyrock** Application
* Bob, the Regional Manager of the supermarket chain - he has several store managers under him:
  * Manager1
  * Manager2
* Charlie, the Head of Security of the supermarket chain - he has several store detectives under him:
  * Detective1
  * Detective2

The following people at `example.com`  have signed up for accounts, but have no reason to be granted access

* Eve - Eve the Eavesdropper
* Mallory - Mallory the malicious attacker
* Rob - Rob the Robber


| Name       |eMail                       |Password | UUID                                  |
|------------|----------------------------|---------|---------------------------------------|
| alice      | alice-the-admin@test.com   | `test`  |`aaaaaaaa-good-0000-0000-000000000000` |
| bob        | bob-the-manager@test.com   | `test`  |`bbbbbbbb-good-0000-0000-000000000000` |
| charlie    | charlie-security@test.com  | `test`  |`cccccccc-good-0000-0000-000000000000` |
| manager1   | manager1@test.com          | `test`  |`manager1-good-0000-0000-000000000000` |
| manager2   | manager2@test.com          | `test`  |`manager2-good-0000-0000-000000000000` |
| detective1 | detective1@test.com        | `test`  |`secure01-good-0000-0000-000000000000` |
| detective2 | detective2@test.com        | `test`  |`secure02-good-0000-0000-000000000000` |
| eve        | eve@example.com            | `test`  |`eeeeeeee-evil-0000-0000-000000000000` |
| mallory    | mallory@example.com        | `test`  |`mmmmmmmm-evil-0000-0000-000000000000` |
| rob        | rob@example.com            | `test`  |`rrrrrrrr-evil-0000-0000-000000000000` |


Two organizations have also been set up by Alice:

| Name       | Description                         | UUID                                 |
|------------|-------------------------------------|--------------------------------------|
| Security   | Security Group for Store Detectives |`security-0000-0000-0000-000000000000`|
| Management | Management Group for Store Managers |`managers-0000-0000-0000-000000000000`|

To save time, the data creating users and organizations from the [previous tutorial](https://github.com/Fiware/tutorials.Identity-Management) has been downloaded and is automatically persisted to the MySQL
database on start-up so the asigned UUIDs do not change and the data does not need to be entered again.

Appropriate roles and permissions have been set up as well.


To refresh your memory about how to create users and organizations, you can log in at `http://localhost:3005/idm`
using the account `alice-the-admin@test.com` with a password of `test`.

![](https://fiware.github.io/tutorials.Roles-Permissions/img/log-in.png)

and look at the organizations list.

To check the roles and permissions, you can select the FIWARE tutorial app and click on **Manage Roles**

If you now log in using other accounts such as Bob, manager1 or Eve you will find you have different levels of access.

* Alice ` alice-the-admin@test.com` can do everything
* Bob `bob-the-manager@test.com` can add Users and Organizations to the Tutorial Application, and add members
* Manager1 `manager1@test.com` can view the Tutorial Application
* Eve `eve@example.com ` has no access to the Tutorial Application





---

## License

[MIT](LICENSE) © FIWARE Foundation e.V.









