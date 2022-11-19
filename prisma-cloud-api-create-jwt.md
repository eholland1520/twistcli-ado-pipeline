#### Appendix: Create a JWT
[Detailed instructions can be found here](https://knowledgebase.paloaltonetworks.com/KCSArticleDetail?id=kA14u0000004MQyCAM&lang=en_US%E2%80%A9&refURL=http%3A%2F%2Fknowledgebase.paloaltonetworks.com%2FKCSArticleDetail)
```
curl -X POST \
            https://api.prismacloud.io/login \
            -H 'Content-Type: application/json' \
            -d '{"username":"Access-Key","password":"Secret-key"}'
```
The command above outputs a JWT that can be used for authenticating to the Prisma Cloud API when downloading the twistcli binary and accessing image scans.
```
{
           "token": "<JWT>",
           "message": "login_successful",
           "customerNames": [
           {
          "customerName": "Test",
          "tosAccepted": true
          }
         ]
        }
```
