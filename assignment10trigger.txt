trigger sendMailToManagerOnUpdate on User (after update) {
String uId;
String mngId;
for(User u:Trigger.new){
if(u.ManagerId!=null){
mngId = u.ManagerId;
uId=u.Id;
}
}
List<Messaging.SingleEmailMessage> emailList= new List<Messaging.SingleEmailMessage>();
List<Account> accList =[Select Id,Name,OwnerId,(Select Id from Contacts) from Account where OwnerId=:uId];
if(accList.size()>0){
Integer noOfAccounts =accList.size();
if(mngId!=null){
User mngObj=[Select Id,Email from User where Id=:mngId];
if(mngObj.Email!=null){
Messaging.SingleEmailMessage mail = new Messaging.SingleEmailMessage();
mail.setSenderDisplayName('Salesforce Manager');
mail.setUseSignature(false);
mail.setBccSender(false);
mail.setSaveAsActivity(false);
mail.toAddresses = new String[]{mngObj.Email};
mail.setSubject('Your Accounts and Number of Related Contacts');
String body = 'Dear User, <br/>';
body += 'You have assigned number of account and Account Contains Number of Contacts.<br/>';
body += 'Total Accounts = '+noOfAccounts;
for(Account accObj : accList){
body += '<br/>'+accObj.Name+' = '+accObj.Contacts.size();
}
mail.setHtmlBody(body);
emailList.add(mail);
}
}
}
if(emailList.size()>0){
Messaging.SendEmailResult[] results = Messaging.sendEmail(emailList);
if (results[0].success)
{
System.debug('The email was sent successfully.');
} else {
System.debug('The email failed to send: '+ results[0].errors[0].message);
}
}
}