This is like programming a normal Javascript application, but you will be restricted to the environment of a sandboxed Javascript Virtual Machine; hence you won't have access to certain things. To help interact with "the outside world", some global variables are available to help you:

 - sender: contains the id of the sender that initiated the transaction
 - owner: contains the id of the owner of the Smart Contract
 - db: contains the tools to interact with the database