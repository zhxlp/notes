# ScriptRunner For Confluence

## 创建工作日志宏

- Macro code

  ```properties
  "<button onclick=createWorkLog('"+context.getSpaceKey()+"')>创建工作日志</button>"
  ```

- Macro Javascript code

  ```javascript
  function getRootPageId(SpaceKey) {
    var pageId = 0;
    jQuery.ajax({
      type: "GET",
      url: "/pages/children.action",
      dataType: "json",
      data: { spaceKey: SpaceKey, node: "root" },
      async: false,
      success: function (data) {
        pageId = data[0].pageId;
      },
    });
    return pageId;
  }

  function getChildrenPageId(parentId, childrenName) {
    var pageId = 0;
    jQuery.ajax({
      type: "GET",
      url: "/pages/children.action",
      dataType: "json",
      data: { pageId: parentId },
      async: false,
      success: function (data) {
        for (var i = 0; i < data.length; i++) {
          if (data[i].text == childrenName) {
            pageId = data[i].pageId;
          }
        }
      },
    });
    return pageId;
  }

  function getWorkLogTitle(currentDate) {
    var year = currentDate.getFullYear();
    var month = currentDate.getMonth() + 1;
    if (month < 10) {
      month = "0" + month;
    }
    var day = currentDate.getDate();
    if (day < 10) {
      day = "0" + day;
    }
    var title = year + "-" + month + "-" + day;
    return title;
  }
  function createBlankPage(spaceKey, parentId, pageName) {
    var pageId = 0;
    jQuery.ajax({
      type: "GET",
      url: "/plugins/createpage/createandviewpage.action",
      data: {
        templateId: 43450384,
        increment: 14,
        target: "View",
        identIndex: 1,
        parentId: parentId,
        spaceKey: spaceKey,
        createPageId: parentId,
        notificationSuppressed: "false",
        title: pageName,
      },
      async: false,
    });
    pageId = getChildrenPageId(parentId, pageName);
    return pageId;
  }
  function createWorkLog(SpaceKey) {
    var currentDate = new Date();
    var year = currentDate.getFullYear() + "年";
    var month = currentDate.getMonth() + 1 + "月";
    month = year + month;
    var day = currentDate.getDate() + "日";
    var title = getWorkLogTitle(currentDate);
    var url =
      "https://wiki.ikmak.com/pages/createpage-entervariables.action?templateId=38338580&spaceKey=" +
      SpaceKey +
      "&newSpaceKey=" +
      SpaceKey +
      "&title=" +
      title +
      "&fromPageId=";
    /* 
        pageId1: 个人空间页面ID
      */
    var pageId1 = getRootPageId(SpaceKey);
    if (pageId1 <= 0) {
      alert("SpaceKey错误");
      return;
    }
    /* 
        pageId2: 工作日志页面ID
      */
    var pageId2 = getChildrenPageId(pageId1, "工作日志");
    if (pageId2 <= 0) {
      if (confirm("工作日志页面不存在,是否创建?")) {
        pageId2 = createBlankPage(SpaceKey, pageId1, "工作日志");
        if (pageId2 <= 0) {
          alert(
            "创建工作日志页面失败,请检查个人空间中是否已经存在该页面！！！"
          );
          return;
        }
      } else {
        return;
      }
    }
    /* 
        pageId3: 年份页面ID
      */
    var pageId3 = getChildrenPageId(pageId2, year);
    if (pageId3 <= 0) {
      if (confirm(year + "页面不存在,是否创建?")) {
        pageId3 = createBlankPage(SpaceKey, pageId2, year);
        if (pageId3 <= 0) {
          alert(
            "创建" + year + "页面失败,请检查个人空间中是否已经存在该页面！！！"
          );
          return;
        }
      } else {
        return;
      }
    }
    /* 
        pageId4: 月份页面ID
      */
    var pageId4 = getChildrenPageId(pageId3, month);
    if (pageId4 <= 0) {
      if (confirm(month + "页面不存在,是否创建?")) {
        pageId4 = createBlankPage(SpaceKey, pageId3, month);
        if (pageId4 <= 0) {
          alert(
            "创建" + month + "页面失败,请检查个人空间中是否已经存在该页面！！！"
          );
          return;
        }
      } else {
        return;
      }
    }
    /*
        pageId5: 日志页面ID
      */
    var pageId5 = getChildrenPageId(pageId4, title);
    if (pageId5 <= 0) {
      url = url + pageId4;
    } else {
      url = "https://wiki.ikmak.com/pages/editpage.action?pageId=" + pageId5;
    }
    window.location.href = url;
  }

  AJS.toInit(function () {
    AJS.$("#main-content")
      .find("button")
      .addClass("aui-button aui-button-primary");
  });
  ```

- Macro CSS style

  ```css
  #main-content button {
    margin-left: 0px;
  }
  ```
