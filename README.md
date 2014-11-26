Salesforce
==========

# Testing

//This repo has data from Salesforce which powers the SFDC tableau dashboard
install.packages('RPostgresSQL')
install.packages('RForcecom')
library(dplyr)
library(RForcecom)
library(RPostgreSQL)


# connect to salesforce
tryCatch(
{
  sfdc_connection<-rforcecom.login("tableau@adroll.com", "8776CyA7Fuu832We6yv36J1RwrhRnvHEmR6v71BIzZm",
                                   "https://na9.salesforce.com/", "28.0")
},
error = function(e)
{
  stop("Issue connecting to Salesforce")
  quit(status=1)
}
)

# connect to Deliroll

driver <<- dbDriver("PostgreSQL")
Postgres <<- dbConnect(driver, dbname = "prodinado", user = 'vikram', password = 'surnamecuriouslydigwind',
                         host = 'db.deli.adroll.com',port = '15432')


# querying salesforce
SFDC_Accounts <- rforcecom.query(sfdc_connection, 
                        "SELECT AccountId,Organization_EID__c,CreatedDate,Account_Manager_Name__c,Account_Manager__c,Region__c,Account.AM_Team_Manager__c,
                        AM_Team__c,Contracted_Monthly_Budget__c,Monthly_Pace__c,Name FROM Opportunity WHERE Account.AM_Team_Manager__c != ' ' AND
                        Monthly_Pace__c > 0.01 AND Account_Manager_Name__c != 'Delight am' AND Account_Manager_Name__c != 'Mike Howard' AND 
                        Account_Manager_Name__c != 'Sam Gurdus' AND 
                        Account_Manager_Name__c != 'michael Rappaport' AND 
                        Account_Manager_Name__c != 'Matt Sayday' AND 
                        Account_Manager_Name__c != 'Lindsay Corbitt' AND 
                        Account_Manager_Name__c != 'katherine Heusner' AND 
                        Account_Manager_Name__c != 'Delight Sales' AND 
                        Account_Manager_Name__c != 'Chris Nixon'")



Total_Accounts_Tableau <- rforcecom.query(sfdc_connection,
                                   "SELECT Account_Manager_First_Name__c,Account_Spend__c,Account_Status__c,AM_Team_Manager__c 
                                  FROM Account WHERE MTD_Spend__c > 1")

#Creating Accounts Assigned Dataframe
Accounts_Assigned_Tableau <- rforcecom.query(sfdc_connection,
                                   "SELECT AccountId,Organization_EID__c,CreatedDate,Account_Manager_Name__c,Account_Manager__c,Account.AM_Team_Manager__c,
                        AM_Team__c,Contracted_Monthly_Budget__c,Monthly_Pace__c,Name FROM Opportunity WHERE Account_Manager_Name__c != ' ' AND 
                        Monthly_Pace__c >= 0.01 AND Account_Manager_Name__c != 'Delight am' AND Account_Manager_Name__c != 'Mike Howard' AND 
                                   Account_Manager_Name__c != 'Sam Gurdus' AND 
                                   Account_Manager_Name__c != 'michael Rappaport' AND 
                                   Account_Manager_Name__c != 'Matt Sayday' AND 
                                   Account_Manager_Name__c != 'Lindsay Corbitt' AND 
                                   Account_Manager_Name__c != 'katherine Heusner' AND 
                                   Account_Manager_Name__c != 'Delight Sales' AND 
                                   Account_Manager_Name__c != 'Chris Nixon' AND 
Monthly_Pace__c > 0.01 AND Account_Manager_Name__c != ' ' 
AND CloseDate = THIS_MONTH ORDER BY Account_Manager_Name__c ASC NULLS LAST")


#Create Calls Logged
Calls_logged_Tableau <-rforcecom.query(sfdc_connection,"SELECT AccountId,Subject,Completed_By_Team__c,Completed_By__c,CreatedById,CreatedDate,Created_By_Team__c,account.Account_Manager_First_Name__c,
Account.AM_Team_Manager__c
FROM Task WHERE Subject LIKE '%CALL%' AND (Subject != 'Message' OR Subject != 'Reply' OR Subject != 'refund')AND Created_By_Team__c = 'Ops'
AND CreatedDate = THIS_MONTH
ORDER BY Completed_By__c
")


Upsells_Submitted_Tableau <-rforcecom.query(sfdc_connection,"SELECT AccountId,account.AM_Team_Manager__c,account.Account_Manager_First_Name__c,Completed_By__c,CreatedDate,LastModifiedDate,OwnerId,Upsell_Opportunity_Amount__c, what.id
FROM Task WHERE RecordType.DeveloperName = 'Upsell_Task'and (account.AM_Team_Manager__c LIKE '%Luecht%' OR account.AM_Team_Manager__c LIKE '%Corbitt%' 
OR account.AM_Team_Manager__c LIKE '%Lawrence%' OR account.AM_Team_Manager__c LIKE '%McWilliams%' 
OR account.AM_Team_Manager__c LIKE '%Brady')
AND LastModifiedDate = THIS_MONTH")

Upsells_Closed_Tableau <-rforcecom.query(sfdc_connection,"SELECT AccountId,account.AM_Team_Manager__c,account.Account_Manager_First_Name__c,RecordType.DeveloperName,Completed_By__c,CreatedDate,
LastModifiedDate,OwnerId,Upsell_Opportunity_Amount__c,status,what.id,IsClosed
FROM Task WHERE RecordType.DeveloperName = 'Upsell_Task'and (account.AM_Team_Manager__c LIKE '%Luecht%' OR account.AM_Team_Manager__c LIKE '%Corbitt%' 
OR account.AM_Team_Manager__c LIKE '%Lawrence%' OR account.AM_Team_Manager__c LIKE '%McWilliams%' 
OR account.AM_Team_Manager__c LIKE '%Brady')
AND LastModifiedDate = THIS_MONTH")

Yesware_Tableau <-rforcecom.query(sfdc_connection,"SELECT Assigned_To_Manager__c,CreatedDate,Created_By_Team__c,Completed_By__c,Type_Of_Activity__c 
FROM Task WHERE Created_By_Team__c = 'Ops' AND CreatedDate = THIS_MONTH AND Type_Of_Activity__c != 'open'")

#Write to Prodinado
# This script creates the tables which are consumed in tableau

dbWriteTable(connection, 'SFDC_Accounts', SFDC_Accounts, row.names = FALSE)
dbWriteTable(connection, 'Total_Accounts_Tableau', Total_Accounts_Tableau, row.names = FALSE)
dbWriteTable(connection, 'Accounts_Assigned_Tableau', Accounts_Assigned_Tableau, row.names = FALSE)
dbWriteTable(connection, 'Calls_logged_Tableau', Calls_logged_Tableau, row.names = FALSE)
dbWriteTable(connection, 'Upsells_Submitted_Tableau', Upsells_Submitted_Tableau, row.names = FALSE)
dbWriteTable(connection, 'Upsells_Closed_Tableau', Upsells_Closed_Tableau, row.names = FALSE)
dbWriteTable(connection, 'Yesware_Tableau', Yesware_Tableau, row.names = FALSE)


