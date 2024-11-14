# Unity_Learn_3D_Kit
## 鏡頭
![{59DB7D30-447A-48A6-9C8A-110445B1BB75}](https://hackmd.io/_uploads/HygHAAS-ke.png)

### 旋轉
![{430362E7-E4B8-4BE5-BC27-6A6DBE0150C7}](https://hackmd.io/_uploads/BkWcqkU-ye.png)

![{0DCBF796-8023-4AD4-8B74-B500B71087E7}](https://hackmd.io/_uploads/SyuPpkLWkx.png)

[Cinemachine Free Look Camera 官方文檔](https://docs.unity3d.com/Packages/com.unity.cinemachine@2.3/manual/CinemachineFreeLook.html)


## 玩家
### 移動
![{27D5B5C7-5542-47C9-80F0-5C6727933DA7}](https://hackmd.io/_uploads/S1G4nRr-ye.png)
### 跳躍
![{7B40349A-FCAC-41EA-BFE1-0AB50D8A9F1D}](https://hackmd.io/_uploads/Hk-N6CHZke.png)
### 攻擊
![{DEC4FD08-8C67-4AC4-9D5A-895FCE0E1DAD}](https://hackmd.io/_uploads/Bk7t60rZyx.png)

## 取得鍵盤輸入
```csharp=
void Update()
{
    m_Movement.Set(Input.GetAxis("Horizontal"), Input.GetAxis("Vertical"));
    m_Camera.Set(Input.GetAxis("Mouse X"), Input.GetAxis("Mouse Y"));
    m_Jump = Input.GetButton("Jump");

    if (Input.GetButtonDown("Fire1"))
    {
        if (m_AttackWaitCoroutine != null)
            StopCoroutine(m_AttackWaitCoroutine);

        m_AttackWaitCoroutine = StartCoroutine(AttackWait());
    }

    m_Pause = Input.GetButtonDown ("Pause");
}
```

### 為什麼Input.GetAxis("Horizontal")就抓得到鍵盤?
![{3E504221-74E1-4EBD-9056-1E20335D36AB}](https://hackmd.io/_uploads/ryUTHbLWkl.png)

[Input Manager官方文檔](https://docs.unity3d.com/Manual/class-InputManager.html)

### 怎麼看程式碼?
- 查看定義：按F12 / 右鍵Go to definition，來查看變數、class在哪裡宣告的
- 查看引用：按Shft+F12 / 右鍵Find Usage，來看哪些地方有用過這個變數

![{EDCA675C-2778-41D6-AD45-8282B9C05ABD}](https://hackmd.io/_uploads/ByCm4lI-1g.png)
▲ 對 m_Movement 按F12找到他是Vector2

![{D07EEEA1-6D01-42C0-A83F-5C597A516D96}](https://hackmd.io/_uploads/BJwJqg8-1l.png)
▲ 對 m_Movement 按Shft+F12找到哪幾行有用到這個變數

![{2A6BCCE4-AA85-477F-9727-FB5AF9D41E21}](https://hackmd.io/_uploads/BJ8a9xUWyx.png)
▲ 找到 MoveInput 會返回 m_Movement

![{4D9536C0-B977-4A09-994E-C65D564EA1F2}](https://hackmd.io/_uploads/ryjGse8Wkx.png)
▲ 對 MoveInput 按Shft+F12找到哪幾行有用到這個變數
→ PlayerController使用了MoveInput來計算移動方向



---

## 操作人物
找到從PlayerInput取得了鍵盤輸入，PlayerController再拿來計算方向。
先全部摺疊起來，從標題看看大致有哪些功能

```csharp=
public class PlayerController : MonoBehaviour, IMessageReceiver
{
    ......

    public void SetCanAttack(bool canAttack)

    // 醒來
    void Awake()

    // 啟用
    void OnEnable()

    // 停用
    void OnDisable()

    // 更新
    void FixedUpdate()

    // Called at the start of FixedUpdate to record the current state of the base layer of the animator.
    void CacheAnimatorState()

    // Called after the animator state has been cached to determine whether this script should block user input.
    void UpdateInputBlocking()

    // Called after the animator state has been cached to determine whether or not the staff should be active or not.
    bool IsWeaponEquiped()

    // Called each physics step with a parameter based on the return value of IsWeaponEquiped.
    void EquipMeleeWeapon(bool equip)

    // Called each physics step.
    void CalculateForwardMovement()

    // Called each physics step.
    void CalculateVerticalMovement()

    // Called each physics step to set the rotation Ellen is aiming to have.
    void SetTargetRotation()

    // Called each physics step to help determine whether Ellen can turn under player input.
    bool IsOrientationUpdated()

    // Called each physics step after SetTargetRotation if there is move input and Ellen is in the correct animator state according to IsOrientationUpdated.
    void UpdateOrientation()

    // Called each physics step to check if audio should be played and if so instruct the relevant random audio player to do so.
    void PlayAudio()

    // Called each physics step to count up to the point where Ellen considers a random idle.
    void TimeoutToIdle()

    // Called each physics step (so long as the Animator component is set to Animate Physics) after FixedUpdate to override root motion.
    void OnAnimatorMove()

    // This is called by an animation event when Ellen swings her staff.
    public void MeleeAttackStart(int throwing = 0)

    // This is called by an animation event when Ellen finishes swinging her staff.
    public void MeleeAttackEnd()

    // This is called by Checkpoints to make sure Ellen respawns correctly.
    public void SetCheckpoint(Checkpoint checkpoint)

    // This is usually called by a state machine behaviour on the animator controller but can be called from anywhere.
    public void Respawn()

    protected IEnumerator RespawnRoutine()

    // Called by a state machine behaviour on Ellen's animator controller.
    public void RespawnFinished()

    // Called by Ellen's Damageable when she is hurt.
    public void OnReceiveMessage(MessageType type, object sender, object data)

    // Called by OnReceiveMessage.
    void Damaged(Damageable.DamageMessage damageMessage)

    // Called by OnReceiveMessage and by DeathVolumes in the scene.
    public void Die(Damageable.DamageMessage damageMessage)
}
```
眼睛很花對吧? 表示塞太多東西了。
最好盡量細分功能，一個程式只負責一件事情
腦袋也會比較分的清楚👍

*總歸一句話，不需要給別人知道的東西，就不要公開出去*

## 生命週期
![image](https://hackmd.io/_uploads/HkFwlzI-1x.png)
### 常用的事件函数(依執行順序)
1. Awake：醒來（一定會做一次，不會重複執行）
2. OnEnable：啟用
3. Start：開始（"啟用後"做一次，不會重複執行）
4. Update：更新（每一幀呼叫一次，啟用後會一直重複）
5. OnDisable：停用
6. OnDestroy：刪除

比喻人來說：起床→有沒有班?→腦袋開機→開始工作->關機->下班



[事件函数的执行顺序](https://docs.unity3d.com/cn/2018.4/Manual/ExecutionOrder.html)

所以主要工作的地方都是在□□□Update
打開FixedUpdate區塊：

```csharp=180
// Called automatically by Unity once every Physics step.
void FixedUpdate()
{
   CacheAnimatorState();//儲存動畫狀態

   UpdateInputBlocking();//更新輸入偵測狀態

   EquipMeleeWeapon(IsWeaponEquiped());//裝備近戰武器

   m_Animator.SetFloat(m_HashStateTime, Mathf.Repeat(m_Animator.GetCurrentAnimatorStateInfo(0).normalizedTime, 1f));//設置動畫時間
   m_Animator.ResetTrigger(m_HashMeleeAttack);//重置攻擊觸發

   if (m_Input.Attack && canAttack)//如果可以攻擊
       m_Animator.SetTrigger(m_HashMeleeAttack);//觸發攻擊動畫

   CalculateForwardMovement();//計算跑步速度
   CalculateVerticalMovement();//計算降落速度

   SetTargetRotation();//計算轉向

   if (IsOrientationUpdated() && IsMoveInput)//如果動了
       UpdateOrientation();//設定轉向

   PlayAudio();//播放音效

   TimeoutToIdle();//是否播放閒置動畫

   m_PreviouslyGrounded = m_IsGrounded;
}
```
看命名找到計算跑步速度函式 → CalculateForwardMovement()

```csharp=253
void CalculateForwardMovement()
{
    // Cache the move input and cap it's magnitude at 1.
    Vector2 moveInput = m_Input.MoveInput;
    if (moveInput.sqrMagnitude > 1f)
        moveInput.Normalize();

    // Calculate the speed intended by input.
    m_DesiredForwardSpeed = moveInput.magnitude * maxForwardSpeed;

    // Determine change to speed based on whether there is currently any move input.
    float acceleration = IsMoveInput ? k_GroundAcceleration : k_GroundDeceleration;

    // Adjust the forward speed towards the desired speed.
    m_ForwardSpeed = Mathf.MoveTowards(m_ForwardSpeed, m_DesiredForwardSpeed, acceleration * Time.deltaTime);

    // Set the animator parameter to control what animation is being played.
    m_Animator.SetFloat(m_HashForwardSpeed, m_ForwardSpeed);
}
```
重要的是輸入從哪裡來？算完被送到哪裡去？
```csharp=253
void CalculateForwardMovement()
{
    //從PlayInput取得移動輸入
    Vector2 moveInput = m_Input.MoveInput;

   （經過一些處理......）

    //設定到Animator的"ForwardSpeed"變數
    m_Animator.SetFloat(m_HashForwardSpeed, m_ForwardSpeed);
}
```
只有設定動畫人物就會動？！

## BlendTree
BlendTree是一個可以用數值去控制混合動畫的功能
![image](https://hackmd.io/_uploads/H1RXhIDWke.png)
▲ AnimatorController中使用ForwardSpeed的地方
![image](https://hackmd.io/_uploads/ByNYhUwZJg.png)
▲ 依ForwardSpeed的大小決定要用走路還是跑步動畫

![image](https://hackmd.io/_uploads/Skj40IPbkl.png)
打開動畫發現有把Root往前(Z軸)移動了5.32 → 這動畫含有Root Motion
[Animator component](https://docs.unity3d.com/2022.3/Documentation/Manual/class-Animator.html)
![image](https://hackmd.io/_uploads/rJ6KgPwbyx.png)

:::warning
如果遇到EX：怎麼跑起來怪怪的？
:::
## 有效率的Debug方式：直接看到數值，不用自己辛苦推算
1. 顯示到Inspector上
```csharp=
protected float m_ForwardSpeed;// How fast Ellen is currently going along the ground.
public float m_ForwardSpeed;//暫時公開成public
[SerializeField] protected float m_ForwardSpeed;//或是加上[SerializeField] 
```