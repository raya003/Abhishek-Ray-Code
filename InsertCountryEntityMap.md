# Abhishek-Ray-Code - Trigger
/* 
        Author       : Jade Global
        Description  : Update currency code on the Contact based on the Contact Country.
        Created Date : 12-4-2014
        */
        
        trigger InsertCountryEntityMap on Contact (before insert, before update)
        {
        
          if(Trigger.isBefore)
          {
            if(Trigger.isInsert || Trigger.isUpdate)
            {
              CountryCurrencyMap.updateCountryOnContact(Trigger.new);
            }
               
          }
          
        }
        
        Apex Class:
        /* 
        Author       : Jade Global
        Description  : Update currency code on the lead based on the lead country.
        Created Date : 12-10-2014
        */
        
        Public class CountryCurrencyMap {
        
        // A utility method for country currency mapping on Contact.
        Public static void updateCountryOnContact(List<Contact> newContacts){
        List<Contact> cont = new List<Contact>(); 
        //Added for CRMSYS-1213 - Starts
        system.debug('conList' + newContacts);
        set<Id> accountIdSet = new Set<Id>();
        for(Contact con : newContacts) {
            if(con.AccountId != null) {
                accountIdSet.add(con.AccountId);
            } 
        }
        Map<Id, Account> accIdMappedWIthAccRecordType = New Map<Id, Account>([select Id, RecordTypeId from Account where Id IN: accountIdSet]);
        RecordType accEnterpriseRecType = [Select Name, Id from RecordType where Name = 'Enterprise Sales' AND SobjectType = 'Account'];
        //Added for CRMSYS-1213 - Ends
        if ((trigger.isInsert || trigger.isUpdate) && trigger.isBefore )
         {
             
        Map<String,Country_Entity_Map__c> newCountryEntityMap   = new Map<String,Country_Entity_Map__c>();
            
          //Create a map for country code and country.
          for(Country_Entity_Map__c con: [SELECT Id,Name,ISO_Country_Code__c,Enterprise_Currency__c,LDC_Currency__c,Individual_LDC_Entity__c,Enterprise_LDC_Entity__c,Continent__c,Country_Territory__c  FROM Country_Entity_Map__c ])
          {
               newCountryEntityMap.put(con.ISO_Country_Code__c,con);
          }
          
         //Map currency code values for all the insert or update context records.
         if(newCountryEntityMap.size() > 0)
          
          {
             for(Contact updateContact:newContacts)
             
              {    
                if(updateContact.MailingCountryCode != Null && newCountryEntityMap.Containskey(updateContact.MailingCountryCode) && (Trigger.isInsert || Trigger.isUpdate))
                  {
                    
                    Country_Entity_Map__c mapCountry =  newCountryEntityMap.get(updateContact.MailingCountryCode);
                    updateContact.LDC_Currency__c = mapCountry.LDC_Currency__c;
                    updateContact.CurrencyIsoCode = mapCountry.LDC_Currency__c;
                  //  updateContact.Individual_LDC_Entity__c = mapCountry.Individual_LDC_Entity__c;
                  //  updateContact.Enterprise_LDC_Entity__c = mapCountry.Enterprise_LDC_Entity__c;
                         // updateContact.Entity__c = mapCountry.Individual_LDC_Entity__c+'-'+mapCountry.Enterprise_LDC_Entity__c;
                 //   updateContact.Entity__c = mapCountry.Individual_LDC_Entity__c;
                    updateContact.Entity__c = mapCountry.Enterprise_LDC_Entity__c;
                    updateContact.Continent__c = mapCountry.Continent__c;
                    updateContact.Country_Territory__c= mapCountry.Country_Territory__c;
                    //Added for CRMSYS-1213 - Starts
                    //system.debug('test' + accIdMappedWIthAccRecordType);
                    //system.debug('test1' + accIdMappedWIthAccRecordType.get(updateContact.AccountId).RecordTypeId);
                    //system.debug('test1' + accEnterpriseRecType);
                    if(accIdMappedWIthAccRecordType.get(updateContact.AccountId) != null && accIdMappedWIthAccRecordType.get(updateContact.AccountId).RecordTypeId == accEnterpriseRecType.Id) {
                         updateContact.LDC_Currency__c = mapCountry.Enterprise_Currency__c;
                         updateContact.CurrencyIsoCode = mapCountry.Enterprise_Currency__c;
                         //updateContact.Contract_Value__c = mapCountry.Enterprise_Currency__c;
                    }
                    //Added for CRMSYS-1213 - Ends                    
                  }
                  //Added for CRMSYS-1213 - Starts
                  else if (updateContact.MailingCountryCode == Null) {
                      updateContact.LDC_Currency__c = 'USD';
                      updateContact.CurrencyIsoCode = 'USD';
                      //updateContact.Contract_Value__c = 'USD';
                  }
                  //Added for CRMSYS-1213 - Ends  
                    
              }
           }
           
         }
        
        }
        
        // A utility method for country currency mapping on Lead.
        Public static void updateCountryOnLead(List<Lead> newLeads, Map<Id,Lead> LeadOldMap){
        //Create maps for country and currency values.
        Map<String,Country_Entity_Map__c> newCountryEntityMap   =  new Map<String,Country_Entity_Map__c>();          
        Map<String,Country_Entity_Map__c> newCountryCodeEntityMap   = new Map<String,Country_Entity_Map__c>();    
        List<Country_Entity_Map__c> CountryEntityMap = [SELECT Id,Name,ISO_Country_Code__c,LDC_Currency__c, Enterprise_Currency__c FROM Country_Entity_Map__c ]; 
        for(Country_Entity_Map__c CountryValue: CountryEntityMap )
        {
           newCountryEntityMap.put(CountryValue.Name,CountryValue);
           newCountryCodeEntityMap.put(CountryValue.ISO_Country_Code__c, CountryValue);
        } 
        RecordType leadSalesRepRecType = [Select Name, Id from RecordType where Name = 'Sales Rep' AND SobjectType = 'Lead'];
        if ( newLeads.size() > 0){            
            for(Lead LeadCountry:newLeads)
            {
                if(!LeadCountry.isConverted){
                    // Check country values from country and lead country fields and map currency accordingly.
                    if((trigger.isInsert)&&(LeadCountry.Lead_country__c == Null || LeadCountry.Lead_country__c =='')&&(LeadCountry.CountryCode != null && LeadCountry.CountryCode != ''))
                    {
                        if(newCountryCodeEntityMap.containsKey(LeadCountry.CountryCode) && newCountryCodeEntityMap.get(LeadCountry.CountryCode)!= null){
                            if(LeadCountry.RecordTypeId ==leadSalesRepRecType.ID){
                                LeadCountry.CurrencyIsoCode = newCountryCodeEntityMap.get(LeadCountry.CountryCode).Enterprise_Currency__c; 
                            }
                            else {
                                LeadCountry.CurrencyIsoCode = newCountryCodeEntityMap.get(LeadCountry.CountryCode).LDC_Currency__c;
                            }
                        }
                    }
                    else if ((trigger.isInsert)&&(LeadCountry.lead_country__c !=Null && LeadCountry.lead_country__c != '' && newCountryEntityMap.get(LeadCountry.lead_country__c) != null)){
                        if(LeadCountry.RecordTypeId ==leadSalesRepRecType.ID){
                            LeadCountry.CurrencyIsoCode = newCountryCodeEntityMap.get(LeadCountry.CountryCode).Enterprise_Currency__c; 
                        }
                        else {
                            LeadCountry.CurrencyIsoCode = newCountryEntityMap.get(LeadCountry.lead_country__c).LDC_Currency__c;
                        }
                    }
                    else if ( (trigger.isUpdate)&& (LeadCountry.lead_country__c != LeadOldMap.get(LeadCountry.Id).lead_country__c)&& LeadCountry.lead_country__c != null && LeadCountry.lead_country__c != '' && newCountryEntityMap.get(LeadCountry.lead_country__c) != null){
                        if(LeadCountry.RecordTypeId ==leadSalesRepRecType.ID){
                            LeadCountry.CurrencyIsoCode = newCountryCodeEntityMap.get(LeadCountry.CountryCode).Enterprise_Currency__c; 
                        }
                        else {
                            LeadCountry.CurrencyIsoCode = newCountryEntityMap.get(LeadCountry.lead_country__c).LDC_Currency__c;
                        }
                    }
                    else if ( (trigger.isUpdate)&& (LeadCountry.CountryCode!= LeadOldMap.get(LeadCountry.Id).CountryCode)&& LeadCountry.CountryCode!= null && LeadCountry.CountryCode!= '' && newCountryCodeEntityMap.get(LeadCountry.CountryCode) != null){
                        if(LeadCountry.RecordTypeId ==leadSalesRepRecType.ID){
                            LeadCountry.CurrencyIsoCode = newCountryCodeEntityMap.get(LeadCountry.CountryCode).Enterprise_Currency__c; 
                        }
                        else {
                            LeadCountry.CurrencyIsoCode = newCountryCodeEntityMap.get(LeadCountry.CountryCode).LDC_Currency__c;
                        }
                    }
                    else if ( (trigger.isUpdate)&& (LeadCountry.CurrencyIsoCode != LeadOldMap.get(LeadCountry.Id).CurrencyIsoCode )&& LeadCountry.CurrencyIsoCode != null && LeadCountry.CurrencyIsoCode != '' && newCountryCodeEntityMap.get(LeadCountry.CountryCode) != null){
                        if(LeadCountry.RecordTypeId ==leadSalesRepRecType.ID){
                            LeadCountry.CurrencyIsoCode = newCountryCodeEntityMap.get(LeadCountry.CountryCode).Enterprise_Currency__c; 
                        }
                        else {
                            LeadCountry.CurrencyIsoCode = newCountryCodeEntityMap.get(LeadCountry.CountryCode).LDC_Currency__c;
                        }
                    }
                }
                 
            }
        }
    }
}
