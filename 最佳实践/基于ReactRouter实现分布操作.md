## 向导页面需要考虑的技术点

- 使用 URL 进行导航，刷新页面依旧停留在当前页
- 每一步操作的表单内容存放的位置
- 页面状态如何切换

需求：

- 创建账号
- 第一步：验证邮箱（输入邮箱）
- 第二步：账号信息（用户名&密码&生日）
- 第三步：完成（确认信息&提交）

实现：

- 三步是同一个 Form，而不是每一步为一个 Form
- 根据路由，渲染当前步骤
- 根据当前步骤渲染按钮（disable/finish）

坑点：

- 使用嵌套路由，复用公共部分（Layout）

  ```jsx
  <Route path="/step">
    <Step />
  </Route>

  <Step>
    <Route path="/step/1">
      <Step1 />
    </Route>
    <Route path="/step/2">
      <Step2 />
    </Route>
  </Step>
  ```

- 当 `<Redirect>` 使用 `exact` 属性时，必须放在 `<Switch></Switch>`，且放在需要跳转路由前，因为 `<Switch>` 匹配到第一个后会停止匹配：

  ```jsx
  <Redirect exact from="/step" to="/step/1" />
  <Route path="/step">
    <Step />
  </Route>
  ```

- 表单数据管理：将 form 传递给子函数

  ```jsx
  // step.js
  import { Form } from 'antd';
  function Step() {
    const [form] = Form.useForm();
    const StepComponent = () => {
      const StepComponent = getSteps()[getCurrentStep()].component;
      return <StepComponent form={form} />;
    };
    return (
      <Form form={form}>
        <Route path="/step/:stepIndex">
          <StepComponent />
        </Route>
      </Form>
    );
  }
  ```

  ```jsx
  // step3.js
  export default function Step3(props) {
    const { email, username, birthday } = props.form.getFieldsValue(true);
    return (
      <div className="finish-step">
        <ul>
          <li>
            <label>Email:</label>
            <span>{email}</span>
          </li>
          <li>
            <label>用户名:</label>
            <span>{username}</span>
          </li>
          <li>
            <label>生日:</label>
            <span>{birthday ? birthday.format('M月D日') : ''}</span>
          </li>
        </ul>
      </div>
    );
  }
  ```
