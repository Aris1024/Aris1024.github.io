<div style="
  /* 👇 核心魔法：添加一个高级的卡片底座 👇 */
  background: linear-gradient(to bottom right, #ffffff, #faf8f5); /* 微微泛暖的渐变底色，像宣纸 */
  border: 1px solid #f0eeec; /* 极细的边框，增加精致感 */
  border-radius: 20px; /* 大圆角 */
  box-shadow: 0 20px 40px -10px rgba(0,0,0,0.08); /* 弥散的投影，让卡片浮起来 */
  padding: 60px 50px; /* 给内部留出呼吸感（画框的留白） */
  margin: 6vh auto; /* 稍微往下压一点，不要紧贴顶部 */
  
  /* 👇 排版部分 👇 */
  max-width: 950px; 
  display: flex; 
  flex-wrap: wrap; 
  align-items: center; 
  justify-content: space-between; 
  gap: 50px; 
  font-family: 'STXingkai', 'KaiTi', serif;">
  
  <!-- 左侧：诗文部分 -->
  <div style="flex: 1; min-width: 280px; text-align: center;">
    <h1 style="font-weight: normal; margin-bottom: 15px; color: #2c2c2c; font-size: 32px;">《 客 至 》</h1>
    <p style="color: #888; font-size: 15px; letter-spacing: 5px; margin-bottom: 35px; margin-left: 5px;">
      作者：杜甫
    </p>
    <p style="font-size: 20px; line-height: 2.4; color: #4a4a4a; letter-spacing: 2px;">
      舍南舍北皆春水，但见群鸥日日来。<br>
      花径不曾缘客扫，蓬门今始为君开。<br>
      盘飧市远无兼味，樽酒家贫只旧醅。<br>
      肯与邻翁相对饮，隔篱呼取尽馀杯。
    </p>
  </div>

  <!-- 右侧：风景画 -->
  <div style="flex: 1.2; min-width: 320px;">
    <img src="_media/background.png" alt="风景图" style="width: 100%; display: block; border-radius: 12px; box-shadow: 0 10px 25px rgba(0,0,0,0.15);">
  </div>

</div>