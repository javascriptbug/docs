# AJAX 额外功能

- [加载指示器](#loader-stripe)
- [表单验证](#ajax-validation)
    - [抛出验证错误](#throw-validation-exception)
    - [显示错误消息](#error-messages)
    - [指定显示错误字段](#field-errors)
- [加载中按钮](#loader-button)
- [Flash信息](#ajax-flash)
- [用法示例](#usage-example)


使用AJAX框架时，您可以选择指定**extras**后缀，该后缀包括其他样式表和JavaScript文件。这些特性在处理前端CMS页面中的AJAX请求时非常有用。

    {% framework extras %}

<a name="loader-stripe"></a>
## 加载指示器

您应该注意的第一个特性是当AJAX请求运行时显示在页面顶部的加载指示器。指示器有钩子在AJAX框架使用的[全局事件](../ajax/javascript-api#global-events)中。

当AJAX请求启动时，将触发ajaxPromise事件，该事件显示指示器并将鼠标光标置于加载状态。ajaxFail和ajaxDone事件用于检测请求何时完成，指示灯何时再次隐藏。

<a name="ajax-validation"></a>
## 表单验证

您可以在表单上用`data-request-validate`属性以启用验证功能。

    <form
        data-request="onSubmit"
        data-request-validate>
        <!-- ... -->
    </form>

<a name="throw-validation-exception"></a>
### 抛出验证错误

在服务器端AJAX处理程序中，可以使用ValidationException类抛出验证异常，以使字段无效，其中第一个参数是数组。该数组的键为字段名 值为错误信息。


    function onSubmit()
    {
        throw new ValidationException(['name' => 'You must give a name!']);
    }

>**注意**：您还可以传递[validation service]（../services/validation）的实例作为异常的第一个参数。

<a name="error-messages"></a>
### 显示错误消息

在表单中，可以通过对容器元素使用“data-validate-error”属性来显示第一条错误消息。容器中的内容将设置为错误消息，元素将变为可见。

    <div data-validate-error></div>

若要显示多条错误消息，请包含具有“data-message”属性的元素。在本例中，段落标记将被复制，并为存在的每条消息设置内容。

    <div class="alert alert-danger" data-validate-error>
        <p data-message></p>
    </div>

要在AJAX失效上添加自定义类，请钩住“ajaxInvalidField”和“ajaxPromise”JS事件。

    $(window).on('ajaxInvalidField', function(event, fieldElement, fieldName, errorMsg, isFirst) {
        $(fieldElement).closest('.form-group').addClass('has-error');
    });

    $(document).on('ajaxPromise', '[data-request]', function() {
        $(this).closest('form').find('.form-group.has-error').removeClass('has-error');
    });

<a name="field-errors"></a>
### Displaying errors with fields
### 指定字段显示错误

或者，您可以通过定义使用“data-validate-for”属性的元素来显示各个字段的验证消息，并将字段名作为值传递。

    <!-- Input field -->
    <input name="phone" />

    <!-- Validation message for the field -->
    <div data-validate-for="phone"></div>

如果元素为空，则将使用服务器的验证文本填充它。否则，您可以指定任何您喜欢的文本，它将被显示。

    <div data-validate-for="phone">
        Oops.. phone number is invalid!
    </div>

<a name="loader-button"></a>
## 加载中按钮

当任何元素包含“data-attach-loading”属性时，将在AJAX请求期间向其添加CSS类“oc-loading”。此类将使用“：after”CSS选择器在按钮和锚定元素上生成一个*加载微调器-loading spinner*。

    <form data-request="onSubmit">
        <button data-attach-loading>
            Submit
        </button>
    </form>

    <a
        href="#"
        data-request="onDoSomething"
        data-attach-loading>
        Do something
    </a>

<a name="ajax-flash"></a>
## Flash信息

在表单上指定“data-request-flash”属性，以便在成功的AJAX请求上使用flash消息

    <form
        data-request="onSuccess"
        data-request-flash>
        <!-- ... -->
    </form>

结合在事件处理程序中使用“Flash”，请求完成后将显示一条Flash消息。

    function onSuccess()
    {
        Flash::success('You did it!');
    }

要与基于AJAX的flash消息保持一致，可以在页面加载时通过将此代码放在页面或布局中呈现[标准flash消息]（../markup/tag-flash）。

    {% flash %}
        <p
            data-control="flash-message"
            class="flash-message fade {{ type }}"
            data-interval="5">
            {{ message }}
        </p>
    {% endflash %}

<a name="usage-example"></a>
## 用法示例

下面是表单验证的完整示例。它调用“onDoSomething”事件，该事件处理程序触发加载提交按钮，对表单字段执行验证，然后显示一条成功的flash消息。

    <form
        data-request="onDoSomething"
        data-request-validate
        data-request-flash>

        <div>
            <input name="name" />
            <span data-validate-for="name"></span>
        </div>

        <div>
            <input name="email" />
            <span data-validate-for="email"></span>
        </div>

        <button data-attach-loading>
            Submit
        </button>

        <div class="alert alert-danger" data-validate-error>
            <p data-message></p>
        </div>

    </form>

这个AJAX查看客户端发送的POST数据，并将一些规则应用于验证器。如果验证失败，则抛出“ValidationException”，否则返回“Flash：：success”消息。

    function onDoSomething()
    {
        $data = post();

        $rules = [
            'name' => 'required',
            'email' => 'required|email',
        ];

        $validation = Validator::make($data, $rules);

        if ($validation->fails()) {
            throw new ValidationException($validation);
        }

        Flash::success('Jobs done!');
    }
