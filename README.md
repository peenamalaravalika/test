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
      "url": "<% var wql = 'SELECT bgefWorkday, contributors (workdayID, featureType, contributor, contributorRole, cf_ZCFXTNDLRVFullNameOnGer007CeContributor as fullName) FROM xtnd_ger007_app_coveredemployeesid_sjwxnp_ger007Bgefs'; return 'data?query=' + wql.urlEncode(); %>",
      "authType": "isu"
    },
    {
      "name": "selectLookups",
      "baseurlType": "workday-wql",
      "url": "<% var wql = 'SELECT ceReferenceName, ceReferenceValue, ceReferenceId, displayOrder, startDate, endDate FROM xtnd_ger007_app_coveredemployeesid_sjwxnp_ger007CeReferences WHERE ceReferenceName in ('Cycle Status') and isActive=true ORDER BY ceReferenceName, displayOrder\"; return 'data?query=' + wql.urlEncode(); %>",
      "authType": "isu"
    },
    {
      "name": "postFieldValue",
      "baseUrlType": "app",
      "url": "ger007CoveredEvents?bulk=true",
      "authType": "sso",
      "httpMethod": "POST",
      "exclude": "<% !addGrid.anyUpdated() %>",
  "onsend": "<% if (addGrid.anyUpdated()) { var addedData = []; for (var row : addGrid.getRows()) { var addMap = {}; addMap.bgefWorkday = row.childrenMap.addBgef.value; addMap.contributors = []; var contrib = {}; contrib.contributor = row.childrenMap.addContributor.value; contrib.contributorRole = row.childrenMap.addContributorRole.value; addMap.contributors.add(contrib); addedData.add(addMap); } self.data = {}; self.data.data = addedData; return self.data; } else { return null; } %>",
      "failonStatusCodes": [
        { "code": 400 }
      ],
      "code": 403
    }
    ,
    {
      "name": "patchFieldValue",
      "baseUrlType": "app",
      "url": "ger007CoveredEvents",
      "authType": "sso",
      "httpMethod": "PATCH",
      "exclude": "<% !editGrid.anyUpdated() %>",
  "onsend": "<% if (editGrid.anyUpdated()) { var patchData = []; for (var row : editGrid.getRows()) { if (!row.isUpdated()) continue; var patchMap = {}; try { patchMap.bgefWorkday = row.childrenMap.editBgef ? row.childrenMap.editBgef.value : row.bgefWorkday; } catch (e) { patchMap.bgefWorkday = row.bgefWorkday; } patchMap.contributors = []; var contrib = {}; if (row.childrenMap.editWorkdayID && row.childrenMap.editWorkdayID.value) { contrib.workdayID = row.childrenMap.editWorkdayID.value; } contrib.contributor = row.childrenMap.editContributor ? row.childrenMap.editContributor.value : null; contrib.contributorRole = row.childrenMap.editContributorRole ? row.childrenMap.editContributorRole.value : null; patchMap.contributors.add(contrib); patchData.add(patchMap); } self.data = {}; self.data.data = patchData; return self.data; } else { return null; } %>",
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
      "exclude": "<% !editGrid.anyRemoved() %>",
  "onsend": "<% // collect removed rows and return an array of contributor workdayIDs to delete if (editGrid.anyRemoved()) { var deleteData = []; var removedRows = (typeof editGrid.getRemovedRows === 'function') ? editGrid.getRemovedRows() : editGrid.getRows(); for (var row : removedRows) { if (typeof row.isRemoved === 'function' && !row.isRemoved()) continue; var delMap = {}; delMap.contributors = []; var contrib = {}; contrib.workdayID = row.childrenMap.editWorkdayID.value; delMap.contributors.add(contrib); deleteData.add(delMap); } self.data = {}; self.data.data = deleteData; return self.data; } else { return null; } %>",
      "failonStatusCodes": [ { "code": 400 } ],
      "code": 403
    }
  ],
  
  "onLoad": "<% pageVariables.allBgefRoles = selectBgefs.data; pageVariables.bgefList = selectBgefs.data; var _seen = {}; pageVariables.contributorList = []; for (var r : selectBgefs.data) { if (!r.contributors) continue; for (var c : r.contributors) { var id = c.contributor ? c.contributor : (c.workdayID ? c.workdayID : null); var desc = c.fullName ? c.fullName : (c.contributor ? c.contributor : id); if (!id) continue; if (!_seen[id]) { _seen[id] = true; var item = {}; item.id = id; item.descriptor = desc; pageVariables.contributorList.add(item); } } } pageVariables.contributorRoles = selectLookups.data; %>",

  "presentation": {
    "title": { "type": "title", "label": "Manage BG/EF Roles - Edit" },
    "body": {
      "type": "section",
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
              "rows": "<% selectBgefs.data %>",
              "columns": [
                { "type": "column", "columnId": "addBgef", "label": "BG/EF", "cellTemplate": { "type": "instanceList", "id": "addBgef", "binding": "addRow.bgefWorkday", "values": "<% pageVariables.bgefList %>" } },
                { "type": "column", "columnId": "addContributor", "label": "Contributor", "cellTemplate": { "type": "instanceList", "id": "addContributor", "binding": "addRow.contributors[0].contributor", "values": "<% pageVariables.contributorList %>" } },
                { "type": "column", "columnId": "addContributorRole", "label": "Contributor Role", "cellTemplate": { "type": "dropdown", "id": "addContributorRole", "binding": "addRow.contributors[0].contributorRole", "values": "<% pageVariables.contributorRoles %>" } }
              ]
            }
            ,
            {
              "type": "grid",
              "id": "editGrid",
              "rowVariableName": "editRow",
              "rows": "<% selectBgefs.data %>",
              "columns": [
                { "type": "column", "columnId": "editWorkdayID", "label": "Workday ID", "cellTemplate": { "type": "input", "id": "editWorkdayID", "binding": "editRow.contributors[0].workdayID" } },
                { "type": "column", "columnId": "editBgef", "label": "BG/EF", "cellTemplate": { "type": "text", "value": "<% editRow.bgefWorkday %>" } },
                { "type": "column", "columnId": "editContributor", "label": "Contributor", "cellTemplate": { "type": "instanceList", "id": "editContributor", "binding": "editRow.contributors[0].contributor", "values": "<% pageVariables.contributorList %>" } },
                { "type": "column", "columnId": "editContributorRole", "label": "Contributor Role", "cellTemplate": { "type": "dropdown", "id": "editContributorRole", "binding": "editRow.contributors[0].contributorRole", "values": "<% pageVariables.contributorRoles %>" } }
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
