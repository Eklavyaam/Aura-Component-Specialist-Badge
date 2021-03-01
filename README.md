# Step by Step guide for "Aura Component Specialist badge challanges

# Challange 1
**Just follow instruction from challange**
**1. Install package**
**2. Change session setting**

![Session Setting](https://user-images.githubusercontent.com/45493653/109465285-f22fb400-7a35-11eb-9a81-b0b2b914b7bd.PNG)



# Challenge 2

# Install Leaflet in your org as static resource.
**URL - https://leafletjs.com/download.html**
***Download and save the Leaflet.zip file***
**Go to your org, Setup -> Static Resource -> New -> Name: Leaflet , Choose Leaflet.zip file that you downloaded -> Save**

[========]

**1.Create new component named BoatSearchForm**

***BoatSearchForm.cmp :***

```
<!--BoatSearchForm-->
<aura:component controller="BoatTypeController" implements="flexipage:availableForAllPageTypes,flexipage:availableForRecordHome,force:hasRecordId" access="global" >
    <aura:attribute type="BoatType__c[]" name="BoatTypes"/>
    <aura:attribute type="Boolean" name="isNewButtonAvailable" default="false"/>
    <aura:attribute type="BoatType__c" name="selectedBoatType"/>
    <aura:handler name="init" value="{!this}" action="{!c.doInit}"/>
   
    <lightning:layout horizontalAlign="center" verticalAlign="end">         
        <lightning:layoutItem padding="around-small">
            <lightning:select name="selectType" aura:id="boatTypes" label="" variant="label-hidden" onchange="{!c.onboatTypechange}">
                <option value="">-- All Types --</option>
                <aura:iteration items="{!v.BoatTypes}" var="boatType">
                    <option value="{!boatType.Id}" text="{!boatType.Name}"/>
                </aura:iteration>
            </lightning:select>
        </lightning:layoutItem>
       
        <lightning:layoutItem padding="around-small">
            <lightning:button variant="brand" label="Search"/>
            <aura:if isTrue="{!v.isNewButtonAvailable}">
                <lightning:button variant="neutral" label="New" onclick="{!c.createBoat}"/>
            </aura:if>
        </lightning:layoutItem>           
    </lightning:layout>
   
</aura:component>
```
***BoatSearchFormController.JS :***
```
({
    doInit : function(component, event, helper) {
        helper.LoadBoatTypes(component,event);
        var seb=component.get("v.selectedBoatType");
        console.log("Seleted boat in doint :"+seb);
        var isEnabled=$A.get("e.force:createRecord");
        if(isEnabled){
            component.set("v.isNewButtonAvailable",true);
        }
    },
   
    onboatTypechange :function(component, event, helper) {
       
        component.set("v.selectedBoatType", component.find("boatTypes").get("v.value"));
       
    }, 
    createBoat: function(component,event){
       
        var boatTypeId=component.get("v.selectedBoatType");
        var createRecordEvent=$A.get("e.force:createRecord");
        createRecordEvent.setParams({
            "entityApiName":"Boat__c",
            "defaultFieldValues": {'BoatType__c': boatTypeId}
        });
        console.log("boatTypeId"+boatTypeId);
        createRecordEvent.fire();
    },
})
```
***BoatSearchFormHelper.JS :***
```
({
    LoadBoatTypes : function(component,event) {
        var action=component.get("c.getBoatType");
       
        action.setCallback(this,function(response) {
            var state= response.getState();
            if(state==='SUCCESS'){
                component.set("v.BoatTypes",response.getReturnValue());
            }
            else {
                console.log("Failed with state: " + state);
            }
        });
        $A.enqueueAction(action);
       
    },
   
   
})
```

**2.Create Component: BoatSearchResults**  — ( This component does not required any changes for now, Just create it and we will modify it for later challange )


**3.Create Component: BoatSearch**

***BoatSearch.cmp :***

```
<!--BoatSearch-->
<aura:component implements="force:appHostable,flexipage:availableForAllPageTypes,flexipage:availableForRecordHome,force:hasRecordId,force:lightningQuickAction" access="global" >
    <div class="FindBoat">
        <lightning:card title="Find a Boat">
            <c:BoatSearchForm/>
        </lightning:card>
    </div>
   
    <lightning:card title="Matching Boats">
        <c:BoatSearchResults aura:id="boatSearchResultsCmp"/>
    </lightning:card>
   
</aura:component>
```

***BoatSearch.CSS :***
```
.THIS.FindBoat {
    margin-bottom:10px
}
```

**4.Create Apex class name : BoatTypeController**

```
public with sharing class BoatTypeController{
    @AuraEnabled
    public static List<BoatType__c> getBoatType(){
    return [SELECT Id, Name FROM BoatType__c ORDER BY Name];
    }
}
```
[========]

**Go to Lightning App builder and create new-> App Page-> Label: Friends with Boats -> Main Region and Right Sidebar** (make sure Label name is correct, copy paste it from challange page if you want.)

![1](https://user-images.githubusercontent.com/45493653/109465264-e93ee280-7a35-11eb-9618-40dcac08ce10.PNG)

**Drag Boatsearch Component to the left side**

![2](https://user-images.githubusercontent.com/45493653/109465266-eb08a600-7a35-11eb-92ad-f07a3e2bad46.PNG)

**Save -> Activation -> Lightning Experience -> Sales -> Add page to app -> Save      Save again.**

![3](https://user-images.githubusercontent.com/45493653/109465772-af221080-7a36-11eb-926e-765c4a00873b.PNG)

**After following above steps ,The sales app should look like below :**

![4](https://user-images.githubusercontent.com/45493653/109466073-235cb400-7a37-11eb-830d-e2954d90160f.PNG)


[========]

**5 Create a Lightning app in developer console: FriendswithBoats**

***FriendswithBoats.app:***

```
<aura:application extends="force:slds">
   <lightning:layout horizontalAlign="center">     
    <c:BoatSearch/>
    </lightning:layout>
</aura:application>
```

**Challange complete.**


[========]

# Challenge 3 & 4

**Here we will update code for both challange. It will pass both.**

[========]

**1.Create Component: BoatTile**

***BoatTile.cmp:***
```
<aura:component implements="flexipage:availableForAllPageTypes,flexipage:availableForRecordHome,force:hasRecordId" access="global" >
    <aura:attribute name="boat" type="Boat__c[]"/>
    <aura:handler name="init" value="{!this}" action="{!c.doInit}"/>
    <lightning:button class="tile">
        <!-- Image -->
        <div style="{!'background-image: url(\'' + v.boat.Picture__c + '\')'}" class="innertile">
            <div class="lower-third">
                <h1 class="slds-truncate">{!v.boat.Name}</h1>
            </div>
        </div>
    </lightning:button>
</aura:component>
```
***BoatTileController.JS:***
```
({
      doInit : function(component, event, helper) {
            var data=component.get("v.boat");
        console.log("data received in boattile"+data);
      }
})
```
***BoatTile.CSS :***
```
.THIS {
   
}
.THIS.tile {
    position:relative;
    display: inline-block;
    width: 220px;
    height: 220px;
    padding: 1px !important ;
    margin-bottom:5px;
}

.THIS .innertile {
    background-size: cover;
    background-position: center;
    background-repeat: no-repeat;
    width: 100%;
    height: 100%;
  
}

.THIS .lower-third {
    position: absolute;
    bottom: 0;
    left: 0;
    right: 0;
    color: #FFFFFF;
    background-color: rgba(0, 0, 0, .4);
    padding: 6px 8px;
   
}

.THIS.tile.selected {
    border:3px solid rgb(0, 112, 210);
       
}
```

**2.create Lightning event : Formsubmit**
```
Formsubmit.evt:
 <aura:event type="COMPONENT" description="Event template">
<aura:attribute name="formData" type="object"/>
</aura:event>
```

**3.Create Apex class : BoatSearchResults**

***BoatSearchResults.Apxc***
```
public class BoatSearchResults {
   
    @AuraEnabled
    public static List<BoatType__c> getBoatType() {
       
        return [select id,Name from BoatType__c];
    } 
   
    @AuraEnabled
    public static List<Boat__c> getBoats(String boatTypeId){
        if(boatTypeId==null || boatTypeId==''){
            return [select id,Name,BoatType__c,Contact__c,Picture__c from Boat__c];  
        }else
        {
        return [select id,Name,BoatType__c,Contact__c,Picture__c from Boat__c where BoatType__c=:boatTypeId];       
        }
       
    }
}
```

**4.Modify component: BoatSearchForm**

***BoatSearchForm.cmp***
```
<!--BoatSearchForm-->
<aura:component controller="BoatSearchResults" implements="flexipage:availableForAllPageTypes,flexipage:availableForRecordHome,force:hasRecordId" access="global" >
    <aura:attribute type="BoatType__c[]" name="BoatTypes"/>
    <aura:attribute type="Boolean" name="isNewButtonAvailable" default="false"/>
    <aura:attribute type="BoatType__c" name="selectedBoatType"/>
    <aura:handler name="init" value="{!this}" action="{!c.doInit}"/>
    <aura:registerEvent name="formsubmit" type="c:formsubmit"/>
   
   
    <lightning:layout horizontalAlign="center" verticalAlign="end">         
        <lightning:layoutItem padding="around-small">
            <lightning:select name="selectType" aura:id="boatTypes" label="" variant="label-hidden" onchange="{!c.onboatTypechange}">
                <option value="">-- All Types --</option>
                <aura:iteration items="{!v.BoatTypes}" var="boatType">
                    <option value="{!boatType.Id}" text="{!boatType.Name}"/>
                </aura:iteration>
            </lightning:select>
        </lightning:layoutItem>
       
        <lightning:layoutItem padding="around-small">
            <lightning:button variant="brand" label="Search" onclick="{!c.onFormSubmit}"/>
           
            <aura:if isTrue="{!v.isNewButtonAvailable}">
                <lightning:button variant="neutral" label="New" onclick="{!c.createBoat}"/>
            </aura:if>
        </lightning:layoutItem>           
    </lightning:layout>
   
</aura:component>
```

***BoatSearchFormController.JS***
```
({
    doInit : function(component, event, helper) {
        //Get the boat Types
        helper.LoadBoatTypes(component,event);
        var seb=component.get("v.selectedBoatType");
        console.log("Seleted boat in doint :"+seb);
        var isEnabled=$A.get("e.force:createRecord");
        if(isEnabled){
            component.set("v.isNewButtonAvailable",true);
        }
    },
   
    onboatTypechange :function(component, event, helper) {
       
        component.set("v.selectedBoatType", component.find("boatTypes").get("v.value"));
       
    }, 
    createBoat: function(component,event){
       
        var boatTypeId=component.get("v.selectedBoatType");
        var createRecordEvent=$A.get("e.force:createRecord");
        createRecordEvent.setParams({
            "entityApiName":"Boat__c",
            "defaultFieldValues": {'BoatType__c': boatTypeId}
        });
        console.log("boatTypeId"+boatTypeId);
        createRecordEvent.fire();
    },
   
    onFormSubmit : function(component,event) {
        var boatTypeId = component.find("boatTypes").get("v.value");
        var data = {
            "boatTypeId" : boatTypeId
        };
       
        var formsubmit = component.getEvent("formsubmit");
        formsubmit.setParams({
            "formData" :data
        });
       
        formsubmit.fire();
    },
})
```

***BoatSearchFormHelper.JS***
```
({
    LoadBoatTypes : function(component,event) {
        var action=component.get("c.getBoatType");
        action.setCallback(this,function(response) {
            var state= response.getState();
            if(state==='SUCCESS'){
                component.set("v.BoatTypes",response.getReturnValue());
            }
            else {
                console.log("Failed with state: " + state);
            }
        });
        $A.enqueueAction(action);  
    },
})
```

**5.Create Component: BoatSearch**

***BoatSearch.cmp***
```
<!--BoatSearch-->
<aura:component implements="force:appHostable,flexipage:availableForAllPageTypes,flexipage:availableForRecordHome,force:hasRecordId,force:lightningQuickAction" access="global" >
    <aura:handler name="formsubmit" event="c:formsubmit" action="{!c.onFormSubmit}"/>
    <div class="FindBoat">
        <lightning:card title="Find a Boat">
            <c:BoatSearchForm/>
        </lightning:card>
    </div>
   
    <lightning:card title="Matching Boats">
        <c:BoatSearchResults aura:id="boatSearchResultsCmp"/>
    </lightning:card>
   
</aura:component>
```

***BoatSearchController.JS***
```
({
    onFormSubmit : function(component, event, helper) {    
        var data=event.getParam("formData");
        var boatSearchResultsCmp = component.find("boatSearchResultsCmp");
        if(boatSearchResultsCmp){
            boatSearchResultsCmp.search(data.boatTypeId);
        }
    }
})
```

***BoatSearch.CSS:***
```
.THIS.FindBoat {
    margin-bottom:10px
}
```

**6.Modify component BoatSearchResults**

***BoatSearchResults.cmp***
```
<!--BoatSearchResults-->
<aura:component controller="BoatSearchResults" implements="force:appHostable,flexipage:availableForAllPageTypes,flexipage:availableForRecordHome,force:hasRecordId,force:lightningQuickAction" access="global" >
   
    <aura:attribute type="Boat__c[]" name="boats"/>
    <aura:attribute type="Boat__c" name="choosenboat"/>
    <aura:handler name="init" value="{!this}" action="{!c.doInit}"/>
    <aura:attribute name="boatTypeId" type="String"/>
   
    <aura:method name="search" action="{!c.doSearch}">
        <aura:attribute name="boatTypeId" type="String"/>
    </aura:method>
   
    <lightning:layout multipleRows="true">
        <aura:if isTrue="{!v.boats.length > 0}">
            <aura:iteration items="{!v.boats}" var="bot">
                <lightning:layoutItem  size="3" flexibility="grow" class="slds-m-around_small">
                    <c:BoatTile boat="{!bot}" />
                </lightning:layoutItem>
            </aura:iteration>
            <aura:set attribute="else">
                <lightning:layoutItem class="slds-align_absolute-center" flexibility="auto" padding="around-small">
                    <ui:outputText value="No boats found" />
                </lightning:layoutItem>
            </aura:set>
        </aura:if>
    </lightning:layout>
</aura:component>
```

***BoatSearchResultsController.JS:***
```
({
    doInit : function(component, event, helper) {  
    },
   
    doSearch : function(component, event, helper) {
        var params = event.getParam('arguments');
        component.set("v.boatTypeId", params.boatTypeId);
                   helper.onSearch(component);
    },
})
```
***BoatSearchResultsControllerHelper.JS***
```
({
    onSearch : function(component,choosenboat,event,helper) {
       
        var action=component.get("c.getBoats");
        action.setParams({"boatTypeId":component.get("v.boatTypeId")});
        action.setCallback(this,function(response) {
            var state= response.getState();
            if(state==='SUCCESS'){ 
                var temp=response.getReturnValue();
                component.set("v.boats",temp);
            }
            else {
                console.log("Failed with state: " + state);
            }
        });
        $A.enqueueAction(action);   
    },
})
```

**Challange complete.**

[========]

# Challenge 5

[========]

**1.Create Lightning event : BoatSelect**
```
BoatSelect.evt:
<aura:event type="APPLICATION" description="Event template">
<aura:attribute name="selectedBoatType" type="BoatType__c"/>
</aura:event>
```
**2.Modify component Boattile**

***BoatTile.cmp:***
```
<aura:component implements="flexipage:availableForAllPageTypes,flexipage:availableForRecordHome,force:hasRecordId" access="global" >
    <aura:attribute name="boat" type="Boat__c[]"/>
    <aura:attribute name="selected" type="Boolean" default="false"/>
    <aura:handler name="init" value="{!this}" action="{!c.doInit}"/>
    <aura:registerEvent name="BoatSelect" type="c:BoatSelect"/>      
    <lightning:button class="{!v.selected ? 'tile selected' : 'tile'}" onclick="{!c.onBoatClick}" value="{!v.boat.Id}">
        <!-- Image -->
        <div style="{!'background-image: url(\'' + v.boat.Picture__c + '\')'}" class="innertile">
            <div class="lower-third">
                <h1 class="slds-truncate">{!v.boat.Name}</h1>
            </div>
        </div>
    </lightning:button>
</aura:component>
```

***BoatTileController.JS:***
```
({
    doInit : function(component, event, helper) {
        var data=component.get("v.boat");
        console.log("data received in boattile"+data);
    },
   
        onBoatClick : function(component, event, helper) {
        var data=event.getSource().get("v.value")
        //var temp=data.id;
        console.log("data received in boattilecontroller"+data);
            var BoatSelect=component.getEvent("BoatSelect");
            BoatSelect.setParams({
                "boatId":data
            });
            BoatSelect.fire();
    }
})
```

***BoatTile.CSS:***
```
.THIS.tile {
    position:relative;
    display: inline-block;
    width: 100%;
    height: 220px;
    padding: 1px !important;
}

.THIS .innertile {
    background-size: cover;
    background-position: center;
    background-repeat: no-repeat;
    width: 100%;
    height: 100%;
}

.THIS .lower-third {
    position: absolute;
    bottom: 0;
    left: 0;
    right: 0;
    color: #FFFFFF;
    background-color: rgba(0, 0, 0, .4);
    padding: 6px 8px;
}
.THIS.selected {
    border: 3px solid rgb(0, 112, 210);
}
```

**3.Modify component: BoatSearchResults**

***BoatSearchResults.cmp:***
```
<!--BoatSearchResults-->
<aura:component controller="BoatSearchResults" implements="force:appHostable,flexipage:availableForAllPageTypes,flexipage:availableForRecordHome,force:hasRecordId,force:lightningQuickAction" access="global" >
   
    <aura:attribute type="Boat__c[]" name="boats"/>
    <aura:attribute type="Boat__c" name="choosenboat"/>
    <aura:handler name="init" value="{!this}" action="{!c.doInit}"/>
    <aura:attribute name="boatTypeId" type="String"/> 
    <aura:attribute name="selectedBoatId" type="String"/>
    <aura:handler name="BoatSelect" event="c:BoatSelect" action="{!c.onBoatSelect}"/>
   
    <aura:method name="search" action="{!c.doSearch}">
        <aura:attribute name="boatTypeId" type="String"/>
    </aura:method>
   
    <lightning:layout multipleRows="true">
        <aura:if isTrue="{!v.boats.length > 0}">
            <aura:iteration items="{!v.boats}" var="boat">
                <lightning:layoutItem  size="3" flexibility="grow" class="slds-m-around_small">
                    <c:BoatTile boat="{!boat}" selected="{!boat.Id == v.selectedBoatId ? true : false }"/>
                </lightning:layoutItem>
            </aura:iteration>
            <aura:set attribute="else">
                <lightning:layoutItem class="slds-align_absolute-center" flexibility="auto" padding="around-small">
                    <ui:outputText value="No boats found" />
                </lightning:layoutItem>
            </aura:set>
        </aura:if>
    </lightning:layout>
</aura:component>
```

***BoatSearchResultsController.JS:***
```
({
    doInit : function(component, event, helper) {  
    },
   
    doSearch : function(component, event, helper) {
        var params = event.getParam('arguments');
        component.set("v.boatTypeId", params.boatTypeId);
                helper.onSearch(component);
    },
    onBoatSelect : function(component, event, helper) {
        var eve = event.getParam('boatId');
        component.set("v.selectedBoatId",eve);
    },
})
```
**Challange complete.**

[========]

# Challenge 6

[========]

**1.Create Lightning event : BoatSelected**

***BoatSelected.evt:***
```
<aura:event type="APPLICATION" description="">
    <aura:attribute name="boat" type="Boat__c"/>
</aura:event>
```

**2.Modify component: BoatTile**

***BoatTile.cmp :***
```
<aura:component implements="flexipage:availableForAllPageTypes,flexipage:availableForRecordHome,force:hasRecordId" access="global" >
    <aura:attribute name="boat" type="Boat__c"/>
    <aura:attribute name="selected" type="Boolean" default="false"/>
    <aura:handler name="init" value="{!this}" action="{!c.doInit}"/>
    <aura:registerEvent name="BoatSelect" type="c:BoatSelect"/>
    <aura:registerEvent name="boatselected" type="c:BoatSelected"/>
   
    <lightning:button class="{!v.selected ? 'tile selected' : 'tile'}" onclick="{!c.onBoatClick}" value="{!v.boat}">
        <!-- Image -->
        <div style="{!'background-image: url(\'' + v.boat.Picture__c + '\')'}" class="innertile">
            <div class="lower-third">
                <h1 class="slds-truncate">{!v.boat.Name}</h1>
            </div>
        </div>
    </lightning:button>
   
</aura:component>
```

***BoatTileController.JS:***
```
({
    doInit : function(component, event, helper) {
        var data=component.get("v.boat");
        console.log("data received in boattile"+data);
    },
   
    onBoatClick : function(component, event, helper) {
        var data=event.getSource().get("v.value")
        var boatid=data.Id;
        console.log("data received in boattilecontroller"+data);
        var BoatSelect=component.getEvent("BoatSelect");
        BoatSelect.setParams({
            "boatId":boatid
        });
        BoatSelect.fire();
       
        //fire BoatSelected event
        var boatselected = $A.get("e.c:BoatSelected");
        boatselected.setParams({
            "boat" : data
        });
        boatselected.fire();
    },
})
```

**3.Create Component: BoatDetails**

***BoatDetails.cmp :***
```
<aura:component implements="flexipage:availableForAllPageTypes,flexipage:availableForRecordHome" access="global" >
    <aura:attribute name="boat" Type="Boat__c"/>
    <aura:attribute name="id" Type="Id"/>
    <aura:handler event="c:BoatSelected" action="{!c.onBoatSelected}"/>
   
    <force:recordData aura:id="service" recordId="{!v.id}"
                      targetFields ="{!v.boat}"
                      fields="Id,
                              Name,
                              Description__c,
                              Price__c,
                              Length__c,
                              Contact__r.Name,
                              Contact__r.Email,
                              Contact__r.HomePhone,
                              BoatType__r.Name,
                              Picture__c"
                      recordUpdated="{!c.onRecordUpdated}"/>
   
    <aura:if isTrue="{! !empty(v.boat) }">
        <lightning:tabset>
            <lightning:tab label="Details">
                <c:BoatDetail boat="{!v.boat}"/>          
            </lightning:tab>
            <lightning:tab label="Reviews">
                Two Content !
            </lightning:tab>
            <lightning:tab label="Add Review">
                Three Content !
            </lightning:tab>
        </lightning:tabset>
    </aura:if>
</aura:component>
```

***BoatDetailsController.JS :***
```
({
    onBoatSelected : function(component, event, helper) {
        console.log("entered boatdetails");
        var data=event.getParam('boat');
        component.set("v.id", data.Id);
        console.log("data received in boatdetails"+data);
        component.find("service").reloadRecord();
    },
   
    onRecordUpdated :function(component, event, helper){
     component.find("service").reloadRecord();  
    }
})
```

**4.Create Component: BoatDetail**  (Earlier we created BoatDetails , this is BoatDetail - no 's' in name)

***BoatDetail.cmp:***
```
<aura:component implements="flexipage:availableForAllPageTypes" access="global" >
    <aura:attribute name="boat" type="Boat__c"/>
    <aura:handler name="init" value="{!this}" action="{!c.init}"/>
    <aura:attribute name="displaybutton" type="Boolean" default="false"/>
    <lightning:card iconName="utility:anchor">
        <aura:set attribute="title">
            {!v.boat.Contact__r.Name}'s Boat
        </aura:set>
        <aura:set attribute="actions">
            <aura:if isTrue="{!v.displaybutton}">
                <lightning:button label="Full Details" onclick="{!c.onFullDetails}" />
            </aura:if>
        </aura:set>
        <lightning:layout horizontalAlign="space" multipleRows="true">
            <lightning:layoutitem size="6" padding="around-small">
                <div class="slds-p-horizontal--small">
                    <div class="boatproperty">
                        <span class="label">Boat Name:{!v.boat.Name}</span>
                        <span></span>
                    </div>
                    <div class="boatproperty">
                        <span class="label">Type:{!v.boat.BoatType__r.Name}</span>
                        <span></span>
                    </div>
                    <div class="boatproperty">
                        <span class="label">Length:{!v.boat.Length__c}</span>
                        <span> ft</span>
                    </div>
                    <div class="boatproperty">
                        <span class="label">Est. Price:</span>
                        <lightning:formattedNumber value="{!v.boat.Price__c}" style="currency" currencyCode="USD"/>
                    </div>
                    <div class="boatproperty">
                        <span class="label">Description:</span>
                        <lightning:formattedRichText value="{! v.boat.Description__c }" />
                    </div>
                </div>
            </lightning:layoutitem>
           
            <lightning:layoutitem size="6" padding="around-small">
               
                <div class="imageview" style="{!'background-image:url(\'' + v.boat.Picture__c + '\')'}" />
            </lightning:layoutitem>
        </lightning:layout>
    </lightning:card>
</aura:component>
```

***BoatDetailController.JS :***
```
({
    init : function(component, event, helper) {
        var redirectToSObjectPageEvent = $A.get("e.force:navigateToSObject");
        if(redirectToSObjectPageEvent){
            component.set("v.displaybutton",true);
        }  
    },
   
    onFullDetails : function(component, event, helper) {
        var redirectToSObjectPageEvent = $A.get("e.force:navigateToSObject");
        var boatrecord = component.get("v.boat");
        if(boatrecord){
            redirectToSObjectPageEvent.setParams({
                "recordId" : boat.Id
            });
            redirectToSObjectPageEvent.fire();
        }
    }
})
```

***BoatDetail.CSS :***
```
.THIS .label {
    font-weight: bold;
    display: block;
}
.THIS.boatproperty {
    margin-bottom: 3px;
}
.THIS .imageview {
    background-repeat: no-repeat;
    background-size: contain;
    height: 200px;
    margin: 2px;  
}
```

[========]

**5.Go to Lightning App builder and select Friends with Boats. Click on edit and add the BoatDetails component in the top right sidebar of the Lightning page.**

![5](https://user-images.githubusercontent.com/45493653/109473444-4a1fe800-7a41-11eb-86b3-87954fdd3f79.PNG)

**Click save. (In preview it will not show anything, but once you save and go to Sales page-> Friends with Boats tab, it will show.)**

![6](https://user-images.githubusercontent.com/45493653/109474647-b8b17580-7a42-11eb-8615-8235e8e5515c.png)

**Challange complete.**

[========]

# Challenge 7

[========]

**1.Create Component: AddBoatReview**

***AddBoatReview.cmp:***
```
<aura:component implements="flexipage:availableForAllPageTypes" access="global" >
    <aura:attribute name="boat" type="Boat__c"/>
    <aura:attribute name="boatReview" type="BoatReview__c"/>
    <aura:attribute name="simpleboatReview" type="BoatReview__c"/>
    <aura:attribute name="recordError" type="String" access="private"/>
    <aura:registerEvent name="BoatReviewAdded" type="c:BoatReviewAdded"/>
    <aura:handler name="init" value="{!this}" action="{!c.doInit}"/>
    <force:recordData aura:id="service" recordId=""
                      targetFields="{!v.boatReview}"
                      fields="Id,Name,Comment__c,Boat__c"
                      targetRecord="{!v.simpleboatReview}"
                      targetError="{!v.recordError}"
                      recordUpdated="{!c.onRecordUpdated}"/>
   
    <div class="slds-form slds-form_stacked">
        <lightning:input aura:id="title" name="Title" label="Title" value="{!v.boatReview.Name}" required="true"/>
        <lightning:inputRichText aura:id="description" title="description" disabledCategories="FORMAT_FONT" value="{!v.boatReview.Comment__c}"/>
        <lightning:button label="Submit" iconName="utility:save" variant="brand" onclick="{!c.onSave}"/>
    </div>
    <aura:if isTrue="{!not(empty(v.recordError))}">
        <div class="recordError">
            {!v.recordError}
        </div>
    </aura:if>
</aura:component>
```

***AddBoatReviewController.JS :***
```
({
    doInit: function(component, event, helper) {
        helper.onInit(component);  
    },
   
   onSave:function(component, event, helper) {
        component.find("service").saveRecord(function(saveResult) {
            if (saveResult.state === "SUCCESS" || saveResult.state === "DRAFT") {
                // record is saved successfully
                var resultsToast = $A.get("e.force:showToast");
                if(resultsToast == undefined){
                    alert("The record was saved.");
                }else{
                    resultsToast.setParams({
                        "title": "Saved",
                        "message": "The record was saved."
                    });
                    resultsToast.fire();
                    helper.onInit(component); 
                    component.getEvent('BoatReviewAdded').fire();
                }
            } else if (saveResult.state === "INCOMPLETE") {
                // handle the incomplete state
                console.log("User is offline, device doesn't support drafts.");
            } else if (saveResult.state === "ERROR") {
                // handle the error state
                console.log('Problem saving contact, error: ' +
                            JSON.stringify(saveResult.error));
            } else {
                console.log('Unknown problem, state: ' + saveResult.state +
                            ', error: ' + JSON.stringify(saveResult.error));
            }
        });     
    },
   onRecordUpdated: function(component, event, helper){
    },
})
```

***AddBoatReviewHelper.JS :***
```
({
    onInit: function(component) {
        component.find("service").getNewRecord(
            "BoatReview__c", // sObject type (entityAPIName)
            null,            // recordTypeId
            false,           // skip cache?
            $A.getCallback( function() {
                var rec=component.get("v.simpleboatReview");
                var error=component.get("v.recordError");
                if(error || (rec === null)) {
                    console.log("Error initializing record template: " + error);
                }
                else { 
                    component.set("v.boatReview.Boat__c", component.get("v.boat.Id"));
                    console.log("Record template initialized: " + rec.apiName);
                }
            })
        );
    } 
})
```

**2.Modify component: BoatDetails**

***BoatDetails.cmp:***
```
<aura:component implements="flexipage:availableForAllPageTypes,flexipage:availableForRecordHome" access="global" >
    <aura:attribute name="boat" Type="Boat__c"/>
    <aura:attribute name="id" Type="Id"/>  
    <aura:attribute access="private" name="selectedTabId" type="String"/>
    <aura:handler event="c:BoatSelected" action="{!c.onBoatSelected}"/>
    <aura:handler event="c:BoatReviewAdded" name="BoatReviewAdded" action="{!c.onBoatReviewAdded}" phase="capture"/>
   
    <force:recordData aura:id="service" recordId="{!v.id}"
                      targetFields ="{!v.boat}"
                      fields="Id,
                              Name,
                              Description__c,
                              Price__c,
                              Length__c,
                              Contact__r.Name,
                              Contact__r.Email,
                              Contact__r.HomePhone,
                              BoatType__r.Name,
                              Picture__c"
                      recordUpdated="{!c.onRecordUpdated}"/>
   
    <aura:if isTrue="{! !empty(v.boat) }">
        <lightning:tabset aura:id="boatTabSet" selectedTabId="{!v.selectedTabId}">
            <lightning:tab label="Details" id="tabDetails">
                <c:BoatDetail boat="{!v.boat}"/>          
            </lightning:tab>
            <lightning:tab label="Reviews" id="boatreviewtab">
                Two Content !
            </lightning:tab>
            <lightning:tab label="Add Review" id="tabAddReview">
                <c:AddBoatReview boat="{!v.boat}"/>
            </lightning:tab>
        </lightning:tabset>
    </aura:if>
</aura:component>
```

***BoatDetailsController.JS:***
```
({
    onBoatSelected : function(component, event, helper) {
        console.log("entered boatdetails");
        var data=event.getParam('boat');
        component.set("v.id", data.Id);
        console.log("data received in boatdetails"+data);
        component.find("service").reloadRecord();
    },
   
    onRecordUpdated :function(component, event, helper){
        component.find("service").reloadRecord();  
    },
   
    onBoatReviewAdded : function(component, event, helper) {
      component.set('v.selectedTabId', 'boatreviewtab');
    },
})
```

**3.Create Lightning event : BoatReviewAdded**

***BoatReviewAdded.evt:***
```
<aura:event type="COMPONENT" description="Event template">
   
</aura:event>
```

**Challange complete.**

[========]

# Challenge 8

[========]

**1.Create Apex class : BoatReviews**

***BoatReviews.apxc:***
```
public class BoatReviews {
    @AuraEnabled
    public static List<BoatReview__c> getAll(Id boatId){
     return [SELECT Id,
	 boat__c,
	 Name,
	 Comment__c,
	 Rating__c,
	 LastModifiedDate,
	 CreatedDate,
	 CreatedBy.Name,
	 CreatedBy.SmallPhotoUrl,
	 CreatedBy.CompanyName 
	 FROM BoatReview__c 
	 WHERE boat__c=:boatId];
    }
}
```

**2.Create Component: BoatReviews**

***BoatReviews.cmp :***
```
<aura:component controller="BoatReviews" implements="flexipage:availableForAllPageTypes" access="global">
    <aura:attribute name="boat" type="Boat__c"/>
    <aura:attribute name="boatReviews" type="BoatReview__c[]" access="private"/>
    <aura:handler name="init" value="{!this}" action="{!c.doInit}"/>
    <aura:handler name="change" value="{!v.boat}" action="{!c.doInit}"/>
    <aura:method name="refresh" action="{!c.doInit}" access="public"/>
    <ui:scrollerWrapper class="slds-feed">
        <ul class="slds-feed__list">
            <aura:iteration items="{!v.boatReviews}" var="boatReview">
                <li class="slds-feed__item">
                    <article class="slds-post">
                        <header class="slds-post__header slds-media">
                            <div class="slds-media__figure">
                                <a href="javascript:void(0);" class="slds-avatar slds-avatar_circle">
                                    <img alt="{!boatReview.CreatedBy.Name}"
                                         src="{!boatReview.CreatedBy.SmallPhotoUrl}"
                                         title="{!boatReview.CreatedBy.Name}" />
                                </a>
                            </div>
                            <div class="slds-media__body">
                                <div>
                                    <p>
                                        <a href="javascript:void(0);"
                                           title="{!boatReview.CreatedBy.Name}"
                                           data-userid="{!boatReview.CreatedBy.Id}"
                                           onclick="{!c.onUserInfoClick}">
                                            {!boatReview.CreatedBy.Name}
                                        </a>
                                        <aura:if isTrue="{!boatReview.CreatedBy.CompanyName}">
                                            — {!boatReview.CreatedBy.CompanyName}
                                        </aura:if>
                                    </p>
                                    <p>
                                        <lightning:formattedDateTime value="{!boatReview.CreatedDate}" />&nbsp;
                                        <lightning:formattedDateTime value="{!boatReview.CreatedDate}" hour="numeric" minute="numeric" second="numeric"/>
                                    </p>
                                </div>
                            </div>
                        </header>
                        <div class="slds-post__content slds-text-longform">
                            <p>{!boatReview.Name}</p>
                            <lightning:formattedRichText value="{!boatReview.Comment__c}" />
                        </div>
                    </article>
                </li>
            </aura:iteration>
        </ul>
        <aura:if isTrue="{!v.boatReviews.length==0}">
            <div class="slds-align_absolute-center">
                No reviews available
            </div>
        </aura:if>
    </ui:scrollerWrapper>
</aura:component>
```

***BoatReviewsController.JS:***
```
({
    doInit : function(component, event, helper) {
        helper.onInit(component);
    },
    onUserInfoClick : function(component, event, helper) {
        var dataUserId = event.target.getAttribute("data-userid");
        var redirectToUserEvent = $A.get("e.force:navigateToSObject");
        if (redirectToUserEvent) {
            redirectToUserEvent.setParams({
                "recordId": dataUserId
            });
            redirectToUserEvent.fire();
        }
    }
})
```

***BoatReviewsHelper.JS :***
```
({
    onInit : function(component, event, helper) {
        var boatId=component.get("v.boat").Id;
        console.log("Boatid received in BoatReviewsHelper :"+boatId);
        var action=component.get("c.getAll");
        action.setParams({"boatId":boatId});
        action.setCallback(this,function(response) {
            var state= response.getState();
            if(state==='SUCCESS'){ 
                var temp=response.getReturnValue();
                component.set("v.boatReviews",temp);
                console.log("BoatReviews received :"+temp);
            }
            else {
                console.log("Failed with state: " + state);
            }
        });
        $A.enqueueAction(action);   
       
    }
})
```

**3.Modify component: BoatDetails**

***BoatDetails.cmp:***
```
<aura:component implements="flexipage:availableForAllPageTypes,flexipage:availableForRecordHome" access="global" >
    <aura:attribute name="boat" Type="Boat__c"/>
    <aura:attribute name="id" Type="Id"/>
    <aura:attribute access="private" name="selectedTabId" type="String"/>
    <aura:handler event="c:BoatSelected" action="{!c.onBoatSelected}"/>
    <aura:handler event="c:BoatReviewAdded" name="BoatReviewAdded" action="{!c.onBoatReviewAdded}" phase="capture"/>
   
    <force:recordData aura:id="service" recordId="{!v.id}"
                      targetFields ="{!v.boat}"
                      fields="Id,
                              Name,
                              Description__c,
                              Price__c,
                              Length__c,
                              Contact__r.Name,
                              Contact__r.Email,
                              Contact__r.HomePhone,
                              BoatType__r.Name,
                              Picture__c"
                      recordUpdated="{!c.onRecordUpdated}"/>
   
    <aura:if isTrue="{! !empty(v.boat) }">
        <lightning:tabset aura:id="boatTabSet" selectedTabId="{!v.selectedTabId}">
            <lightning:tab label="Details" id="tabDetails">
                <c:BoatDetail boat="{!v.boat}"/>          
            </lightning:tab>
            <lightning:tab label="Reviews" id="boatreviewtab">
                <c:BoatReviews aura:id="boatreviewcomponent" boat="{!v.boat}"/>
            </lightning:tab>
            <lightning:tab label="Add Review" id="tabAddReview">
                <c:AddBoatReview boat="{!v.boat}"/>
            </lightning:tab>
        </lightning:tabset>
    </aura:if>
</aura:component>
```

***BoatDetailsController.JS :***
```
({
    onBoatSelected : function(component, event, helper) {
        console.log("entered boatdetails");
        var data=event.getParam('boat');
        component.set("v.id", data.Id);
        console.log("data received in boatdetails"+data);
        component.find("service").reloadRecord();
    },
   
    onRecordUpdated :function(component, event, helper){
        component.find("service").reloadRecord();   
    },
   
    onBoatReviewAdded : function(component, event, helper) {
        component.set('v.selectedTabId', 'boatreviewtab');
        var boatreviewcomponent = component.find("boatreviewcomponent");
        if(boatreviewcomponent){
            boatreviewcomponent.refresh();
        }
    },
})
```

**Challange complete.**

[========]

# Challenge 9

[========]


**1.Create Component: FiveStarRating**


***FiveStarRating.cmp:***
```
<aura:component implements="flexipage:availableForAllPageTypes" access="global" >
    <aura:attribute name="value" type="Integer" default="0"/>
    <aura:attribute name="readonly" type="Boolean" default="false"/>
    
    <ltng:require scripts="{!$Resource.fivestar + '/rating.js'}" 
                  styles="{!$Resource.fivestar + '/rating.css'}" 
                  afterScriptsLoaded="{!c.afterScriptsLoaded}" />
    <aura:handler name="change" value="{!v.value}" action="{!c.onValueChange}"/>
    <ul aura:id="ratingarea" class="{! (readonly) ? 'readonly c-rating' : 'c-rating' }"/>
</aura:component>
```

***FiveStarRatingController.JS:***
```
({

    afterScriptsLoaded : function(component, event, helper) {
        var domEl = component.find("ratingarea").getElement();
        var currentRating = component.get("v.value"); 
        var readOnly = component.get("v.readonly"); 
        var maxRating = 5;
        
        var callback = function(rating) {
            component.set('v.value', rating);
        }
        
        component.ratingObj = rating(domEl, currentRating, maxRating, callback, readOnly); 
    },
    
    onValueChange: function(component,event,helper) {
        if (component.ratingObj) {
            var value = component.get('v.value');
            component.ratingObj.setRating(value,false);
        }
    }
})
```

**2.Modify component: AddBoatReview AND Modify component: BoatReviews** (Modify both)

***AddBoatReview.cmp:***
```
<aura:component implements="flexipage:availableForAllPageTypes" access="global" >
    <aura:attribute name="boat" type="Boat__c"/>
    <aura:attribute name="boatReview" type="BoatReview__c"/>
    <aura:attribute name="simpleboatReview" type="BoatReview__c"/>
    <aura:attribute name="recordError" type="String" access="private"/>
    <aura:registerEvent name="BoatReviewAdded" type="c:BoatReviewAdded"/>
    <aura:handler name="init" value="{!this}" action="{!c.doInit}"/>
    <force:recordData aura:id="service" recordId=""
                      targetFields="{!v.boatReview}"
                      fields="Id,Name,Comment__c,Boat__c"
                      targetRecord="{!v.simpleboatReview}"
                      targetError="{!v.recordError}"
                      recordUpdated="{!c.onRecordUpdated}"/>
    
    <div class="slds-form slds-form_stacked">
        <lightning:input aura:id="title" name="Title" label="Title" value="{!v.boatReview.Name}" required="true"/>
        <lightning:inputRichText aura:id="description" title="description" disabledCategories="FORMAT_FONT" value="{!v.boatReview.Comment__c}"/>
        <c:FiveStarRating value="{!v.boatReview.Rating__c}" readonly="false"/>
        <lightning:button label="Submit" iconName="utility:save" variant="brand" onclick="{!c.onSave}"/>
    </div>
    <aura:if isTrue="{!not(empty(v.recordError))}">
        <div class="recordError">
            {!v.recordError}
        </div>
    </aura:if>
</aura:component>
```

***BoatReviews.cmp:***
```
<aura:component controller="BoatReviews" implements="flexipage:availableForAllPageTypes" access="global">
    <aura:attribute name="boat" type="Boat__c"/>
    <aura:attribute name="boatReviews" type="BoatReview__c[]" access="private"/>
    <aura:handler name="init" value="{!this}" action="{!c.doInit}"/>
    <aura:handler name="change" value="{!v.boat}" action="{!c.doInit}"/>
    <aura:method name="refresh" action="{!c.doInit}" access="public"/>
    <ui:scrollerWrapper class="slds-feed">
        <ul class="slds-feed__list">
            <aura:iteration items="{!v.boatReviews}" var="boatReview">
                <li class="slds-feed__item">
                    <article class="slds-post">
                        <header class="slds-post__header slds-media">
                            <div class="slds-media__figure">
                                <a href="javascript:void(0);" class="slds-avatar slds-avatar_circle">
                                    <img alt="{!boatReview.CreatedBy.Name}" 
                                         src="{!boatReview.CreatedBy.SmallPhotoUrl}" 
                                         title="{!boatReview.CreatedBy.Name}" />
                                </a>
                            </div>
                            <div class="slds-media__body">
                                <div>
                                    <p>
                                        <a href="javascript:void(0);" 
                                           title="{!boatReview.CreatedBy.Name}" 
                                           data-userid="{!boatReview.CreatedBy.Id}" 
                                           onclick="{!c.onUserInfoClick}">
                                            {!boatReview.CreatedBy.Name}
                                        </a>
                                        <aura:if isTrue="{!boatReview.CreatedBy.CompanyName}"> 
                                            — {!boatReview.CreatedBy.CompanyName}
                                        </aura:if>
                                    </p>
                                    <p>
                                        <lightning:formattedDateTime value="{!boatReview.CreatedDate}" />&nbsp;
                                        <lightning:formattedDateTime value="{!boatReview.CreatedDate}" hour="numeric" minute="numeric" second="numeric"/>
                                    </p>
                                </div>
                            </div>
                        </header>
                        <div class="slds-post__content slds-text-longform">
                            <p>{!boatReview.Name}</p>
                            <lightning:formattedRichText value="{!boatReview.Comment__c}" />
                            <c:FiveStarRating value="{!boatReview.Rating__c}" readonly="true"/>
                        </div>
                    </article>
                </li>
            </aura:iteration>
        </ul>
        <aura:if isTrue="{!v.boatReviews.length==0}">
            <div class="slds-align_absolute-center">
                No reviews available
            </div>
        </aura:if>
    </ui:scrollerWrapper>
</aura:component>
```

**Challange complete.**

[========]

# Challenge 10

[========]


**1.Create Lightning event : PlotMapMarker**
```
PlotMapMarker.evt:
<aura:event type="APPLICATION" description="">
    <aura:attribute name="sObjectId" type="String" />
    <aura:attribute name="lat" type="String" />
    <aura:attribute name="long" type="String" />
    <aura:attribute name="label" type="String" />
</aura:event>
```

**2.Modify component: BoatTile**

***BoatTile.cmp:***
```
<aura:component implements="flexipage:availableForAllPageTypes,flexipage:availableForRecordHome,force:hasRecordId" access="global" >
    <aura:attribute name="boat" type="Boat__c"/>
    <aura:attribute name="selected" type="Boolean" default="false"/>
    <aura:handler name="init" value="{!this}" action="{!c.doInit}"/>
    <aura:registerEvent name="BoatSelect" type="c:BoatSelect"/> 
    <aura:registerEvent name="boatselected" type="c:BoatSelected"/>
    <aura:registerEvent name="plotMapMarker" type="c:PlotMapMarker"/>
    <lightning:button class="{!v.selected ? 'tile selected' : 'tile'}" onclick="{!c.onBoatClick}" value="{!v.boat}"> 
        <!-- Image -->
        <div style="{!'background-image: url(\'' + v.boat.Picture__c + '\')'}" class="innertile">
            <div class="lower-third">
                <h1 class="slds-truncate">{!v.boat.Name}</h1>
            </div>
        </div>
    </lightning:button>
</aura:component>
```

***BoatTileController.JS:***
```
({
    doInit : function(component, event, helper) {
        var data=component.get("v.boat");
        console.log("data received in boattile"+data);
    },

    onBoatClick : function(component, event, helper) {
        var data=event.getSource().get("v.value")
        var boatid=data.Id;
        console.log("data received in boattilecontroller"+data);
        var BoatSelect=component.getEvent("BoatSelect");
        BoatSelect.setParams({
            "boatId":boatid
        });
        BoatSelect.fire();
        //fire BoatSelected event
        var boatselected = $A.get("e.c:BoatSelected"); 
        boatselected.setParams({
            "boat" : data
        }); 
        boatselected.fire(); 
        var plotMapMarkerEvent = $A.get("e.c:PlotMapMarker");
        plotMapMarkerEvent.setParams({
            "sObjectId" : selectedBoat.Id,
            "lat" : selectedBoat.Geolocation__Latitude__s,
            "long" : selectedBoat.Geolocation__Longitude__s,
            "label" : selectedBoat.Name
        });
        plotMapMarkerEvent.fire();
    },
})
```


**3.Modify component: Map** (This component will be created automatically after challange 9 submit. we just need to modify it.)



***Map.cmp:***
```
<aura:component  implements="flexipage:availableForAllPageTypes" access="global">
    <aura:attribute name="width"  type="String" default="100%" />
    <aura:attribute name="height" type="String" default="200px" />
    <aura:attribute name="location" type="SObject"/>
    <aura:attribute name="jsLoaded" type="boolean" default="false"/>
    <ltng:require styles="{!$Resource.Leaflet + '/leaflet.css'}"
                  scripts="{!$Resource.Leaflet + '/leaflet-src.js'}"
                  afterScriptsLoaded="{!c.jsLoaded}" /> 
    <aura:handler event="c:PlotMapMarker" action="{!c.onPlotMapMarker}"/>
    <div aura:id="map">
        <lightning:card title="Current Boat Location">
            <div style="{!'width: ' + v.width + '; height: ' + v.height}">
                <div 
                     class="slds-align_absolute-center">
                    Please make a selection
                </div>
            </div>
        </lightning:card>
    </div>
</aura:component>
```


***MapController.JS:***
```
({
    jsLoaded : function(component) {
        component.set("v.jsLoaded", true);
    },
    onPlotMapMarker : function(component, event) {
        var location = component.get("v.location");
        var locationData = {
            sObjectId : event.getParam("sObjectId"),
            lat : event.getParam("lat"),
            long : event.getParam("long"),
            label : event.getParam("label")
        };
        component.set("v.location", locationData);
    } 
})
```

***Map.css:***
```
.THIS {
    width: 100%;
    height: 100%;
}
```

***MapRenderer.js:***
```
({
    rerender: function (component) {
        var nodes = this.superRerender();
        var location = component.get('v.location');
        if (!location) {
        } else {
            // If the Leaflet library is not yet loaded, we can't draw the map: return
            if (!window.L) {
                return nodes;
            }
            // Draw the map if it hasn't been drawn yet
            if (!component.map) {
                var mapElement = component.find("map").getElement();
                component.map = L.map(mapElement, {zoomControl: true}).setView([42.356045, -71.085650], 13);
                component.map.scrollWheelZoom.disable();
                window.L.tileLayer('https://server.arcgisonline.com/ArcGIS/rest/services/World_Street_Map/MapServer/tile/{z}/{y}/{x}', {attribution: 'Tiles © Esri'}).addTo(component.map);
            }
            if (location && location.lat && location.long) {
                var latLng = [location.lat, location.long];
                if (component.marker) {
                    component.marker.setLatLng(latLng);
                } else {
                    component.marker = window.L.marker(latLng);
                    component.marker.addTo(component.map);
                }
                component.map.setView(latLng);
            }
            return nodes;
        }
    }
})
```


***Map.design:***
```
<design:component label="Map">
  <design:attribute name="width" label="Width" />
  <design:attribute name="height" label="Height" />
</design:component>
```

***Map.svg:***
```
<?xml version="1.0" encoding="UTF-8" standalone="no"?>

<svg width="100px" height="100px" viewBox="0 0 100 100" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">

    <!-- Generator: Sketch 39.1 (31720) - http://www.bohemiancoding.com/sketch -->

    <title>Slice</title>

    <desc>Created with Sketch.</desc>

    <defs></defs>

    <g id="Page-1" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">

        <rect id="Rectangle" fill="#62B7ED" x="0" y="0" width="100" height="100" rx="8"></rect>

        <path d="M84.225,26.0768044 L62.925,15.4268044 C61.8895833,14.9830544 60.70625,14.9830544 59.81875,15.4268044 L40.1458333,25.3372211 L20.325,15.4268044 C19.1416667,14.8351377 17.6625,14.8351377 16.6270833,15.5747211 C15.5916667,16.1663877 15,17.3497211 15,18.5330544 L15,71.7830544 C15,73.1143044 15.7395833,74.2976377 16.9229167,74.8893044 L38.2229167,85.5393044 C39.2583333,85.9830544 40.4416667,85.9830544 41.3291667,85.5393044 L61.15,75.6288877 L80.8229167,85.5393044 C81.2666667,85.8351377 81.8583333,85.9830544 82.45,85.9830544 C83.0416667,85.9830544 83.78125,85.8351377 84.3729167,85.3913877 C85.4083333,84.7997211 86,83.6163877 86,82.4330544 L86,29.1830544 C86,27.8518044 85.4083333,26.6684711 84.225,26.0768044 L84.225,26.0768044 Z M78.6041667,32.8809711 L78.6041667,60.9851377 C78.6041667,62.6122211 77.125,63.7955544 75.6458333,63.2038877 C70.1729167,61.1330544 74.6104167,51.9622211 70.6166667,46.9330544 C66.91875,42.3476377 62.1854167,47.0809711 57.6,39.8330544 C53.3104167,32.8809711 59.0791667,27.8518044 64.4041667,25.1893044 C65.14375,24.8934711 65.8833333,24.8934711 66.475,25.1893044 L77.4208333,30.6622211 C78.3083333,31.1059711 78.6041667,31.9934711 78.6041667,32.8809711 L78.6041667,32.8809711 Z M48.8729167,74.0018044 C47.9854167,74.4455544 46.95,74.2976377 46.2104167,73.7059711 C44.73125,72.3747211 43.5479167,70.3038877 43.5479167,68.2330544 C43.5479167,64.6830544 37.63125,65.8663877 37.63125,58.7663877 C37.63125,52.9976377 30.8270833,51.5184711 25.0583333,52.1101377 C23.5791667,52.2580544 22.54375,51.2226377 22.54375,49.7434711 L22.54375,28.1476377 C22.54375,26.3726377 24.31875,25.1893044 25.7979167,26.0768044 L38.51875,32.4372211 C38.6666667,32.4372211 38.8145833,32.5851377 38.8145833,32.5851377 L39.2583333,32.8809711 C44.5833333,35.9872211 43.5479167,38.5018044 41.3291667,42.3476377 C38.8145833,46.6372211 37.7791667,42.3476377 34.2291667,41.1643044 C30.6791667,39.9809711 27.1291667,42.3476377 28.3125,44.7143044 C29.4958333,47.0809711 33.0458333,44.7143044 35.4125,47.0809711 C37.7791667,49.4476377 37.7791667,52.9976377 44.8791667,50.6309711 C51.9791667,48.2643044 53.1625,49.4476377 55.5291667,51.8143044 C57.8958333,54.1809711 59.0791667,58.9143044 55.5291667,62.4643044 C53.4583333,64.5351377 52.5708333,68.9726377 51.6833333,71.9309711 C51.5354167,72.5226377 51.0916667,73.1143044 50.5,73.4101377 L48.8729167,74.0018044 L48.8729167,74.0018044 Z" id="Shape" fill="#FFFFFF"></path>

    </g>

</svg>
```
