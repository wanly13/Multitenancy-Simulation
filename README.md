# Multitenancy-Simulation
Simulación de algunas acciones en un multitenancy

- Se trata de guardar todas los CVs de los trabajadores de cada empresa

1) Se crea un nuevo Bucket S3 "pgs-web-multi-frontend" y una carpeta
 "empresas-suscritas" 
 -Se añaden 2 empresas como suscritas QAM y OBCA

2) Se crea un nuevo TEMA SNS "nuevos-cvs",  y adicionalmente se crea un lamda "leer-pdfs" 
 
```
#Leer pdf
import json
import boto3

def lambda_handler(event, context):
    # Entrada (json)
    archivo_id = event['Records'][0]['s3']['object']['key']
    tenant_id = archivo_id.split('/')[1] # Empresas suscritas
    archivo_last_modified = event['Records'][0]['eventTime']
    archivo_size = event['Records'][0]['s3']['object']['size']
    
    archivo = {
        'tenant_id': tenant_id,
        'archivo_id': archivo_id,
        'archivo_datos': {
            'last_modified': archivo_last_modified,
            'size': archivo_size,
            
        }    
    }
    # Publicar en SNS
    sns_client = boto3.client('sns')
    response_sns = sns_client.publish(
        TopicArn = 'arn:aws:sns:us-east-1:426340658243:nuevos-cvs',
        Subject = 'Nuevo Archivo',
        Message = json.dumps(archivo),
        MessageAttributes = {
            'tenant_id': {'DataType': 'String', 'StringValue': tenant_id }
        }
    )    
    # TODO implement
    return {
        'statusCode': 200,
        'body': response_sns
    }
```



Creamos las bases de datos para cada empresa 



creamos el lambda para guardar en OBCA

```
# Empresa : OBCA
import json
import boto3
def lambda_handler(event, context):
    
    # Entrada (json)
    print(event)
    body = json.loads(event['Records'][0]['body'])
    Message = json.loads(body['Message'])
    # Proceso
    dynamodb = boto3.resource('dynamodb')
    table = dynamodb.Table('t_multi_OBCA')
    archivo = {
        'tenant_id': Message['tenant_id'],
        'archivo_id': Message['archivo_id'],
        'archivo_datos': Message['archivo_datos']
    }
    print(archivo) # Revisar en CloudWatch
    response = table.put_item(Item=archivo)
    # Salida (json)
    return {
        'statusCode': 200,
        'response': response
    }
```

creamos el lambda para guardar en OBCA

```
# Empresa : QAM
import json
import boto3
def lambda_handler(event, context):
    
    # Entrada (json)
    print(event)
    body = json.loads(event['Records'][0]['body'])
    Message = json.loads(body['Message'])
    # Proceso
    dynamodb = boto3.resource('dynamodb')
    table = dynamodb.Table('t_multi_QAM')
    archivo = {
        'tenant_id': Message['tenant_id'],
        'archivo_id': Message['archivo_id'],
        'archivo_datos': Message['archivo_datos']
    }
    print(archivo) # Revisar en CloudWatch
    response = table.put_item(Item=archivo)
    # Salida (json)
    return {
        'statusCode': 200,
        'response': response
    }
```


