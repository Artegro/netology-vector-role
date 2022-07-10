# Домашнее задание к занятию "7.6. Написание собственных провайдеров для Terraform."

Бывает, что 
* общедоступная документация по терраформ ресурсам не всегда достоверна,
* в документации не хватает каких-нибудь правил валидации или неточно описаны параметры,
* понадобиться использовать провайдер без официальной документации,
* может возникнуть необходимость написать свой провайдер для системы используемой в ваших проектах.   

## Задача 1. 
Давайте потренируемся читать исходный код AWS провайдера, который можно склонировать от сюда: 
[https://github.com/hashicorp/terraform-provider-aws.git](https://github.com/hashicorp/terraform-provider-aws.git).
Просто найдите нужные ресурсы в исходном коде и ответы на вопросы станут понятны.  


1. Найдите, где перечислены все доступные `resource` и `data_source`, приложите ссылку на эти строки в коде на 
гитхабе.   
1. Для создания очереди сообщений SQS используется ресурс `aws_sqs_queue` у которого есть параметр `name`. 
    * С каким другим параметром конфликтует `name`? Приложите строчку кода, в которой это указано.
    * Какая максимальная длина имени? 
    * Какому регулярному выражению должно подчиняться имя? 

## Ответ 1
### 1. все доступные `resource` и `data_source` 

   * Карта `data_source`с перечислением всех исходящих данных начинается с [426 строки](https://github.com/hashicorp/terraform-provider-aws/blob/main/internal/provider/provider.go#L426) 
     и идет до  [918 строки](https://github.com/hashicorp/terraform-provider-aws/blob/main/internal/provider/provider.go#L918) 
   * Карта же ресурсов `resource` начинается сразу дарее с [920 строки](https://github.com/hashicorp/terraform-provider-aws/blob/main/internal/provider/provider.go#L920)
      и идет до [2102 строки](https://github.com/hashicorp/terraform-provider-aws/blob/main/internal/provider/provider.go#L2102)

-----------
  
### 2.  Для создания очереди сообщений SQS используется ресурс `aws_sqs_queue` у которого есть параметр `name`.
  * Параметр `name` конфликтует как мы видим из кода с "name_prefix" [Строка 87](https://github.com/hashicorp/terraform-provider-aws/blob/main/internal/service/sqs/queue.go#L87)
    
   ``` bash
    		"name": {
			  Type:          schema.TypeString,
  			Optional:      true,
  			Computed:      true,
  			ForceNew:      true,
	  		ConflictsWith: []string{"name_prefix"},
   ```
  * Какая максимальная длина имени определена в [строке 427](https://github.com/hashicorp/terraform-provider-aws/blob/main/internal/service/sqs/queue.go#L427) 80 символов
    
   ```bash
    } else {
			re = regexp.MustCompile(`^[a-zA-Z0-9_-]{1,80}$`)
		}
   ```
  * Какому регулярному выражению должно подчиняться имя?  
    Так же становиться понятно из ответа выше  выражение 
   
   ```bash
   re = regexp.MustCompile(`^[a-zA-Z0-9_-]{1,80}$`)
   
   ```
   [Строка 427](https://github.com/hashicorp/terraform-provider-aws/blob/main/internal/service/sqs/queue.go#L427) Говорит о том, что должны бить большие и малые буквы и цифры  

-----------

## Задача 2. (Не обязательно) 
В рамках вебинара и презентации мы разобрали как создать свой собственный провайдер на примере кофемашины. 
Также вот официальная документация о создании провайдера: 
[https://learn.hashicorp.com/collections/terraform/providers](https://learn.hashicorp.com/collections/terraform/providers).

1. Проделайте все шаги создания провайдера.
2. В виде результата приложение ссылку на исходный код.
3. Попробуйте скомпилировать провайдер, если получится то приложите снимок экрана с командой и результатом компиляции.   

