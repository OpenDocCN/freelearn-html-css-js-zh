# 第五章. 动态 Visualforce 绑定

动态 Visualforce 绑定是 Spring '11 版本中最大的特性之一。我们可以使用这个特性来构建通用的 Visualforce 页面，而无需考虑页面上必须显示哪些记录字段。记录字段是在运行时而不是编译时确定的。这是一个强大的特性，它允许我们最小化代码（Visualforce 和 Apex 代码）。否则，我们必须编写更多的查询，填充记录列表，并渲染更多的字段。使用动态 Visualforce 绑定，我们可以开发一个页面，根据不同用户的授权或偏好以不同的方式渲染。

本章涵盖以下主题：

+   使用标准对象和自定义对象的动态引用

+   引用 Apex Maps 和 Lists

+   与字段集一起工作

+   动态引用全局变量

让我们了解动态 Visualforce 绑定...

# 使用标准对象和自定义对象的动态引用

Salesforce 支持标准对象和自定义对象的动态 Visualforce 绑定。我们可以使用以下形式的动态绑定：

`reference[expression]`

让我们详细讨论前面的形式：

+   `reference`：这可以是一个 sObject、Apex 类或全局变量。

+   `expression`：这可以是字段名称或相关对象。如果是相关对象，则可以使用递归选择的字段或进一步相关对象。

### 小贴士

动态绑定可以在公式表达式有效的页面上使用。它使用`{!}`符号。如果从 Apex 类中引用，则特定的属性（sObject 或变量）必须是公共的或全局的。

### 注意

**定义关系**：如果在表达式中需要评估对象关系，它们会变得复杂。考虑我们的例子，其中`Order__c`自定义对象与`Customer__c`自定义对象有关联。这两个对象之间的关系称为`Orders__r`。`Customer__c`对象具有`Email__c`字段。以下动态类型转换的查找将返回相同的`Email__c`字段：

+   `Order__c.Customer__c['Email__c']`

+   `Order__c['Customer__c.Email__c']`

+   `Order__c['Customer__c']['Email__c']`

+   `Order__c.Orders__r[Email__c]`

+   `Order__c[Orders__r.Email__c]`

+   `Order__c[Orders__r][Email__c]`

动态 Visualforce 页面必须有一个标准控制器，并且进一步的实现可以在关联的控制器扩展中完成。原因是 Visualforce 自动处理页面`StandardController`或`StandardSetController`对象执行的 SOQL 查询的优化，只加载使用的字段。

当我们创建具有静态引用的页面时，页面可以在编译期间识别字段和对象。然后`StandardController`对象将特定的字段和对象转换为 SOQL 查询。但是，动态引用是在运行时而不是在编译时评估的。这意味着动态引用是在执行`StandardController`对象的 SOQL 查询之后评估的。因此，当我们使用动态引用并需要向控制器扩展提供一些额外信息时，我们可以使用`addFields()`方法添加任意数量的额外字段。此方法将传递一个额外字段列表给`StandardController`，这些字段将加载而不会产生运行时错误。`addField()`方法的用法如下：

```js
public DynamicOrderExtension(ApexPages.StandardController controller) {         
controller.addFields(editableFields);
}
```

以下示例展示了动态 Visualforce 绑定的用法。此页面显示了一个带有一些可编辑字段的订单记录。一些字段与`object(Customer__c)`相关。我们可以通过对象关系遍历来理解动态引用的用法。

```js
<apex:page standardController="Order__c" extensions="DynamicOrderExtension">
    <apex:pageMessages /><br/>
    <apex:form >
        <apex:pageBlock title="Edit Order" mode="edit">        
            <apex:pageBlockSection columns="1">
                <apex:inputField value="{!Order__c.Name}"/>
                <apex:repeat value="{!editableFields}" var="f">
                    <apex:inputField value="{!Order__c[f]}"/>
                </apex:repeat>
            </apex:pageBlockSection>
        </apex:pageBlock>
    </apex:form>
</apex:page>
```

以下代码是前面 Visualforce 页面的控制器扩展。`DynamicOrderExtension`控制器扩展有一个名为`editableFields`的字符串列表，这个字符串列表包含`Order__c`对象的一些字段名以及`Order__c`的相关对象（`Customer__c`）的一些字段。在这个例子中，可编辑字段是硬编码的。但我们可以通过使用 Apex 的`Schema.sObjectType`方法来获取动态引用的信息。这将使引用更加动态和强大。例如，`Schema.SobjectType.Order__c.fields.getMap()`返回一个包含`Order__c`字段名的映射。前面的标记中包含`<apex:repeat>`标签，用于循环`editableFields`字符串列表，以及`<apex:inputField>`标签，用于显示返回的特定字符串。它表示订单字段和相关对象字段的名字。下面的标记行显示了动态引用：

```js
<apex:inputField value="{!Order__c[f]}"/>

public with sharing class DynamicOrderExtension {
    public final Order__c orderDetails { get; private set; }

    public DynamicOrderExtension(ApexPages.StandardController controller) { 
        String qid = ApexPages.currentPage().getParameters().get('id');
        String theQuery = 'SELECT Id, ' + joinList(editableFields, ', ') +
        ' FROM Order__c WHERE Id = :qid';
        this.orderDetails = Database.query(theQuery);
        controller.addFields(editableFields);
    }

    public List<String> editableFields {
        get {
            if (editableFields == null) {
                editableFields = new List<String>();
                editableFields.add('Delivered__c');
                editableFields.add('Customer__c');
                editableFields.add('Planned_Delivery_Date__c');
                editableFields.add('Status__c');
                editableFields.add('Customer__r.Email__c');
            }
            return editableFields ;
        }
        private set;
    }

    private static String joinList(List<String> theList, String separator) {

        if (theList == null) {
            return null;
        }

        if (separator == null) {
            separator = '';
        }

        String joined = '';
        Boolean firstItem = true;

        for (String item : theList) {
            if(null != item) {
                if(firstItem){
                    firstItem = false;
                }
                else {
                    joined += separator;
                }
                joined += item;
            }
        }
        return joined;
    }
}
```

此页面需要通过指定为 id 查询参数的有效案例记录 ID 来访问。例如，[`c.ap1.visual.force.com/apex/DynamicBindingExample?id=a02900000086Hlr`](https://c.ap1.visual.force.com/apex/DynamicBindingExample?id=a02900000086Hlr)。

# 引用 Apex Maps 和 Lists

Apex Maps 和 Lists 可以在 Visualforce 页面中动态引用。Apex Lists 与`<apex:pageBlockTable>`和`<apex:repeat>`标签广泛使用。在我们的前面例子（在*使用标准对象和自定义对象的动态引用*部分）中，我们已经看到了 Apex List 的动态引用。以下示例展示了 Apex Map 的动态引用。这是`DynamicExampleListMap`页面的标记：

```js
<apex:page controller="DynamicBindingsMapListExample">
  <apex:form >
    <apex:actionFunction name="reDisplayCustomers" rerender="cust" />
    <apex:pageBlock title="Criteria">
       <apex:outputLabel value="Starting Letter"/>
       <apex:selectList value="{!selectedKey}" size="1" onchange="reDisplayCustomers()">
          <apex:selectOptions value="{!keys}" />
       </apex:selectList>
    </apex:pageBlock>
    <apex:pageBlock title="Customers">

             <apex:outputPanel id="cust">
                <apex:pageBlockTable value="{!customerMap[selectedKey]}" var="cus">
                   <apex:column value="{!cus.name}"/>
                   <apex:column value="{!cus.Address__c}"/>
                   <apex:column value="{!cus.Email__c}"/>

                </apex:pageBlockTable>
             </apex:outputPanel>
    </apex:pageBlock>
  </apex:form>
</apex:page>
```

以下是与之相关的自定义控制器。`customerMap`对象包含客户记录，下拉列表会根据 Map 中的适当值动态填充。我们可以从下拉列表和客户列表中选择一个字母，然后根据所选字母重新排列结果。`customerMap`对象通过使用`{!customerMap[selectedKey]}`动态引用在运行时返回相应的客户列表：

```js
public class DynamicBindingsMapListExample
{
    public Map<string, List<Customer__c>> customerMap{get; set;}
    public List<selectoption> keys {get; set;}
    public String selectedKey {get;set;}
    public Map<string, Customer__c> custByName {get;set;}

    public Set<string> getMapKeys()
    {
        return customerMap.keySet();
    }

    public DynamicBindingsMapListExample()
    {
        custByName = new Map<string, Customer__c>();
        List<string> sortedKeys=new List<string>();
        customerMap = new Map<string, list<Customer__c>>();
        customerMap.put('All', new List<Customer__c>());
        List<Customer__c> customers = [SELECT Id, Name, Email__c, Address__c FROM Customer__c ORDER BY Name asc];

        for (Customer__c tempCustomer : customers)
        {
            customerMap.get('All').add(tempCustomer);
            String start = tempCustomer.Name.substring(0,1);
            List<Customer__c> custFromMap = customerMap.get(start);
            if (custFromMap == null)
            {
                custFromMap =new List<Customer__c>();
                customerMap.put(start, custFromMap);
            }
            custFromMap.add(tempCustomer);
            custByName.put(tempCustomer.name, tempCustomer);
        }

        keys=new List<selectoption>();
        for (String key : customerMap.keySet())
        {
            if (key != 'All')
            {
                sortedKeys.add(key);
            }
        }
        sortedKeys.sort();
        sortedKeys.add(0, 'All');

        for (String key : sortedKeys)   {
            keys.add(new SelectOption(key, key));
        }         
        selectedKey='All';
    }
}
```

# 与字段集一起工作

字段集是一组可以以声明方式定义的字段。字段集在 API 版本 21.0 的 Visualforce 页面上可用。这些字段集可以通过动态绑定在 Visualforce 页面上显示。例如，假设我们创建了一个字段集（字段集名称：`CustomerDetails`），其中包含客户对象的`Email__c`、`Name`和`Address__c`字段。我们可以在 Visualforce 中如下引用`CustomerDetails`字段集：

```js
<apex:page standardController="Customer__c">
    <apex:repeat value="{!$ObjectType.Customer__c.FieldSets.CustomerDetails}" var="f">
        <apex:outputText value="{! Customer__c [f]}" /><br/>
    </apex:repeat>
</apex:page>
```

当我们想要创建一个管理包或在字段集中添加/删除/重新排序字段时，我们可以不修改任何代码就完成这些操作。

### 小贴士

一个 Visualforce 页面最多可以有 50 个字段集。

# 摘要

在本章中，我们了解了 Spring'11 发布的一项强大功能——动态绑定。我们熟悉了标准对象和自定义对象动态引用的用法。我们还对引用 Apex Maps/List 以及使用字段集的方式有了很好的了解。我们还看到了全局变量动态引用的用法。通过所有这些，我们学习了最小化 Visualforce 和 Apex 代码的机制。
