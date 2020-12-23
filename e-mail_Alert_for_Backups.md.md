# Alertas de e-mail con SNS para los diferentes estados de los jobs de AWS Backups

Se accede al servicio ``AWS Backups > Backups plans > Create Backup plan`` y se crea un plan de copias de seguridad 

![create_backup_plan](image/create_backup_plan.png)

Se elige la opción `Build a new plan` y se asigna un nombre. 

![name_backup_plan](image/name_backup_plan.PNG)

En la misma página en la parte inferior, se va a configurar el `Backup rule`. Las reglas de las copias de seguridad es para indicar cada cuanto tiempo hay que realizar las copias de seguridad.

Se le asigna un nombre y se define la regla de respaldo con ``backup schedule``, ``backup window`` y ``lifecycle rules``.

![backup_rule](image/backup_rule.PNG)

Se deja el `Backup Vault` por defecto y creamos el plan.

Lo siguiente es añadir los recursos que se quiera realizar la zopia de seguridad. Para esto en se accede a `AWS Backup > Backup plans > testBackupPlan > Assign resources`.

![assign_resources](image/assign_resources.png)

Se le asigna el nombre y en `Assign by` se elige `Resource ID`, el tipo de recurso y el id del recurso.

![assign_resources2](image/assign_resources2.PNG)

Ahora se va a crear un `Topics` con el servicio de ***SNS**. Se accede a `Amazon SNS > Topics > Create topics` 

![topic](image/topic.png)

Se asigna el nombre del topic y el tipo ``Standard``.

![create_topic](image/create_topic.PNG)

Se añade una política de acceso:

![access_policy](image/access_policy.PNG)

Hay que añadir el arn del topic creado anteriormente y la id del propietario. Esta id se puede conseguir en `Amazon SNS > Topics > testBackupTopic`.

El código es el siguiente:
~~~~
{
  "Version": "2008-10-17",
  "Id": "__default_policy_ID",
  "Statement": [
    {
      "Sid": "__default_statement_ID",
      "Effect": "Allow",
      "Principal": {
        "AWS": "*"
      },
      "Action": [
        "SNS:Publish",
        "SNS:RemovePermission",
        "SNS:SetTopicAttributes",
        "SNS:DeleteTopic",
        "SNS:ListSubscriptionsByTopic",
        "SNS:GetTopicAttributes",
        "SNS:Receive",
        "SNS:AddPermission",
        "SNS:Subscribe"
      ],
      "Resource": "arn:aws:sns:eu-west-1:600239822244:testBackupTopic",
      "Condition": {
        "StringEquals": {
          "AWS:SourceOwner": "600239822244"
        }
      }
    },
    {
      "Sid": "__console_pub_0",
      "Effect": "Allow",
      "Principal": {
        "Service": "backup.amazonaws.com"
      },
      "Action": "SNS:Publish",
      "Resource": "arn:aws:sns:eu-west-1:600239822244:testBackupTopic"
    }
  ]
}
~~~~

Lo sigueinte es crear una subcripción, para acceder en `Amazon SNS > Topics > testBackupTopic`.

![subscription](image/subscription.png)

El arn se asigna solo, se elige el tipo de protocolo, en este caso ``Email`` y el `Endpoint` que es el punto final donde acabrá llegando las alertas, nuestro correo.

![create_subscription](image/create_subscription.PNG)

Si el protocolo es de ``Email``, hay que confirmar la subcripción en un correo de confirmación.

![confirm_subscription](image/confirm_subscription.PNG)

Por último tenemos que ejecutar el siguiente comando en `AWS cli`.

~~~~
aws backup put-backup-vault-notifications \
--backup-vault-name Default \
--sns-topic-arn arn:aws:sns:eu-west-1:600239822244:testBackupTopic \
--backup-vault-events BACKUP_JOB_STARTED BACKUP_JOB_COMPLETED BACKUP_JOB_FAILED 
~~~~