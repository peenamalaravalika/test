{
  "id": "ger007ceidAdminManageBgefRolesEdit",
  "_securityDomains": [
    "Ger007ManageCeIdentification"
  ],
  "endPoints": [
    {
      "name": "currentuser",
      "baseurlType": "workday-common",
      "url": "workers/me",
      "authType": "isu"
    },
    {
      "name": "selectBgefs",
      "baseurlType": "workday-wql",
      "url": "<% 
	    var wql = 'SELECT bgefWorkday, contributors (workdayID, featureType, contributor, contributorRole, cf_ZCFXTNDLRVFullNameOnGer007CeContributor as fullName)' +
           		  'FROM xtnd_ger007_app_coveredemployeesid_sjwxnp_ger007Bgefs';
				  
	    return 'data?query=' + wql.urlEncode(); 
	   %>",
      "authType": "isu"
    },
	"outboundData" : {
	"outboundEndPoints" : [
    {
      "name": "postFieldValue",
      "baseUrlType": "app",
      "url": "ger007CoveredEvents?bulk=true",
      "authType": "sso",
      "httpMethod": "POST",
      "exclude": "<% !addGrid.anyUpdated() %>",
      "onsend": "<%
	    if (addGrid.anyUpdated()) 
		{
		  var addedData = [];
		  
		  for (var row : addGrid.getRows())
		  {
		    var addMap = {:};
			addMap.bgefWorkday = row.childrenMap.addBgef.value;
			addMap.contributors = row.childrenMap.addContributor.value;
			addMap.contributorRole = row.childrenMap.addContributorRole.value;
			addedData.add(addMap);
			}
			
			self.data = {:};
			self.data.data = addedData;

			return self.data;
		} 
		else 
		{
		  return null;
		}
	  %>",
      "failonStatusCodes": [
        { 
		  "code": 400 
		},
        {
          "code": 403
        }
	  ]
	 },
    {
      "name": "patchFieldValue",
      "baseUrlType": "app",
      "url": "ger007CoveredEvents",
      "authType": "sso",
      "httpMethod": "PATCH",
      "exclude": "<% !editGrid.anyUpdated() %>",
  "onsend": "<%\nif (editGrid.anyUpdated()) {\n  var patchData = [];\n  for (var row : editGrid.getRows()) {\n    // only include rows that were updated\n    if (!row.isUpdated()) continue;\n    var patchMap = {};\n    // include BG/EF if available on the row (bind to either the input or the source value)\n    try { patchMap.bgefWorkday = row.childrenMap.editBgef ? row.childrenMap.editBgef.value : row.bgefWorkday; } catch (e) { patchMap.bgefWorkday = row.bgefWorkday; }\n    patchMap.contributors = [];\n    var contrib = {};\n    // include workdayID when present (for updates), otherwise omit to indicate create-like behavior\n    if (row.childrenMap.editWorkdayID && row.childrenMap.editWorkdayID.value) { contrib.workdayID = row.childrenMap.editWorkdayID.value; }\n    contrib.contributor = row.childrenMap.editContributor ? row.childrenMap.editContributor.value : null;\n    contrib.contributorRole = row.childrenMap.editContributorRole ? row.childrenMap.editContributorRole.value : null;\n    patchMap.contributors.add(contrib);\n    patchData.add(patchMap);\n  }\n  self.data = {};\n  self.data.data = patchData;\n  return self.data;\n} else {\n  return null;\n}\n%>",
      "failonStatusCodes": [ { "code": 400 } ],
      "code": 403
    }
    ,
    {
      "name": "deleteFieldValue",
      "baseUrlType": "app",
      "url": "ger007CoveredEvents",
      "authType": "sso",
      "httpMethod": "DELETE",
      "exclude": "<% ABC %>",
      "failonStatusCodes": [ 
	    { 
		  "code": 400
		},
		{
          "code": 403
        }
	   ]
	  }
  "onLoad": "<% pageVariables.allBgefRoles = getAllBgefRoles(); %>",
  "presentation": {
    pageType": "Edit",
    "title": {
	"type": "title",
	"label": "Identify BG/EF Roles" 
	},
    "body": {
      "type": "section",
	  "horizontal"
      "children": [
        {
          "type": "section",
          "children": [
            { "type": "button", "id": "backToGrid", "label": "Back", "action": "LINK", "taskReference": { "taskId": "ger007ceidAdminManageBgefRoles" } }
          ]
        },
        {
          "type": "section",
          "children": [
            {
              "type": "grid",
              "id": "addGrid",
              "rowVariableName": "addRow",
              "rows": "<% pageVariables.allBgefRoles %>",
              "columns": [
                { "type": "column", "columnId": "addBgef", "label": "BG/EF", "cellTemplate": { "type": "instanceList", "id": "addBgef", "binding": "addRow.bgefWorkday" } },
                { "type": "column", "columnId": "addContributor", "label": "Contributor", "cellTemplate": { "type": "instanceList", "id": "addContributor", "binding": "addRow.contributors[0].contributor" } },
                { "type": "column", "columnId": "addContributorRole", "label": "Contributor Role", "cellTemplate": { "type": "dropdown", "id": "addContributorRole", "binding": "addRow.contributors[0].contributorRole" } }
              ]
            }
            ,
            {
              "type": "grid",
              "id": "editGrid",
              "rowVariableName": "editRow",
              "rows": "<% pageVariables.allBgefRoles %>",
              "columns": [
                { "type": "column", "columnId": "editWorkdayID", "label": "Workday ID", "cellTemplate": { "type": "input", "id": "editWorkdayID", "binding": "editRow.contributors[0].workdayID" } },
                { "type": "column", "columnId": "editBgef", "label": "BG/EF", "cellTemplate": { "type": "text", "value": "<% editRow.bgefWorkday %>" } },
                { "type": "column", "columnId": "editContributor", "label": "Contributor", "cellTemplate": { "type": "instanceList", "id": "editContributor", "binding": "editRow.contributors[0].contributor" } },
                { "type": "column", "columnId": "editContributorRole", "label": "Contributor Role", "cellTemplate": { "type": "dropdown", "id": "editContributorRole", "binding": "editRow.contributors[0].contributorRole" } }
              ],
              "editable": true
            }
          ,
          {
            "type": "section",
            "id": "pageFooter",
            "children": [
              { "type": "text", "value": "\u00A9 ${currentYear} My Company — Manage BG/EF Roles" }
            ]
          }
          ]
        }
      ]
    }
  }
}
