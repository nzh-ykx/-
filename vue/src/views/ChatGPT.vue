<!-- views/chatgpt.vue -->

<template>
  <div class="common-layout">
    <el-container>
      <el-header style=" background: linear-gradient(blue, pink);">

        <div style="display: flex;justify-content: center;align-items: center;">
          <div style="width: 20%;font-size: 30px;margin-top: 10px;"></div>
          <div style="width: 60%;font-size: 30px;margin-top: 10px;">电脑配置助手</div>
          <!-- <image src="../../public/a" style="width: 50px;height: 50px; border-radius: 50%;"/> -->
          <div style="width: 20%;font-size: 30px;margin-top: 10px;"></div>
        </div>
      </el-header>
      <el-container>
        <el-aside style="background-color:#00FFFF;text-align: center;">
          <!--  <div style="width: 15px;margin: 100px auto; font-size: 30px; ">
                输入文字点击提交对 AI 进行提问
            </div> -->
          <div
              style="border-radius: 25%;    width: 100%;height: 100px;background-color: 	#7FFFAA;margin-bottom: 5px;display: flex;justify-content: center;align-items: center;">
            <text style="font-size: 40px;">智能问答</text>
          </div>
          <div
              style=" border-radius: 25%;   width: 100%;height: 100px;background-color: 	#7FFFAA;margin-bottom: 5px;display: flex;justify-content: center;align-items: center;">
            <text style="font-size: 40px;">历史记录</text>
          </div>
          <div
              style="border-radius: 25%;   width: 100%;height: 100px;background-color: 	#7FFFAA;margin-bottom: 5px;display: flex;justify-content: center;align-items: center;">
            <text style="font-size: 40px;">使用指南</text>
          </div>
          <!-- 			<div style="width: 100%;height: 100px;background-color: green;margin-bottom: 5px;display: flex;justify-content: center;align-items: center;"><text style="font-size: 40px;"></text></div>
                      <div style="width: 100%;height: 100px;background-color: green;margin-bottom: 5px;display: flex;justify-content: center;align-items: center;"><text style="font-size: 40px;"></text></div> -->
        </el-aside>
        <el-container>
          <el-main style="background-color:#FFFFFF;height:500px">

            <el-descriptions column=1 size="small" border="true" style="width:1000px;">
              <div v-for="item in chatRecord" :key="item">
                <el-descriptions-item :label="item.sender" label-align="center">
                  <span v-html="item.message"></span>
                </el-descriptions-item>
              </div>
            </el-descriptions>
          </el-main>
          <el-footer style="padding:10px;height:400px;">

            <div style="width:1000px; height: 200px;">
              <div style="display: flex;justify-content: center;align-items: center; margin-bottom: -50px;">
                <div style="width:333px; height: 100px;">
                  <el-input v-model="work" type="text" placeholder="请输入您的职业"
                            :autosize="{ minRows: 5, maxRows: 7 }" resize="none" show-word-limit maxlength="10"/>
                </div>
                <div style="width:333px; height: 100px;">
                  <el-input v-model="maxPrice" type="text" placeholder="请输入最大价格"
                            :autosize="{ minRows: 5, maxRows: 7 }" resize="none" show-word-limit maxlength="10"/>
                </div>
                <div style="width:333px; height: 100px;">
                  <el-input v-model="minPrice" type="text" placeholder="请输入最小价格"
                            :autosize="{ minRows: 5, maxRows: 7 }" resize="none" show-word-limit maxlength="10"/>
                </div>
              </div>
              <el-input v-model="question" type="textarea" placeholder="请输入你的需求"
                        :autosize="{ minRows: 5, maxRows: 7 }" resize="none" show-word-limit maxlength="300"/>
              <div style="text-align: right;">
                <el-button color="#8d3d45" @click="chat" plain style="margin: 5px 0;width:100px;" size="large"
                           :disabled="click">提交
                </el-button>
              </div>
            </div>
          </el-footer>
        </el-container>
      </el-container>
    </el-container>
  </div>

</template>

<script>

export default {
  name: 'ChatGPT',
  data() {
    return {
      work: "",
      minPrice: "",
      maxPrice: "",
      question: "",
      count: 0,
      click: false,
      chatRecord: [{sender: "AI:", message: "您好，主人！有什么需要向我提问的吗？"}]
    }
  },
  methods: {
    chat() {
      this.click = true;
      //this.question = encodeURIComponent(this.question); // 对用户输入进行URL编码
      let url = `http://localhost:9090/getComputer?work=${this.work}&minPrice=${this.minPrice}&maxPrice=${this.maxPrice}&requirement=${this.question}`;

      fetch(url, {
        method: 'GET',
        headers: {
          'Accept': 'application/json' // 告诉服务器我们期望接收JSON格式的响应
        }
      }).then(response => {
            if (!response.ok) {
              throw new Error('Network response was not ok');
            }
            return response.json(); // 解析JSON格式的响应
          })
          .then(data => {
            this.updateChatRecord(data); // 处理数据并更新聊天记录
          })
          .catch(error => {
            console.error('There has been a problem with your fetch operation:', error);
            this.click = false; // 请求失败时重置点击状态
          });
    },
    updateChatRecord(data) {
      // 清空当前AI的消息，准备接收新的回复
      let ai = this.chatRecord[this.chatRecord.length - 1];
      ai.message = ""; // 清空加载提示
      // 遍历返回的电脑列表并构建消息
      data.computerList.forEach(computer => {
        ai.message += `
		方案: ${computer.planID}<br>
        处理器: ${computer.cpu}<br>
        处理器价格: ${computer.cpuPrice}<br>
        主板: ${computer.motherBorad}<br>
        主板价格: ${computer.motherBoradPrice}<br>
        机箱: ${computer.case1}<br>
        机箱价格: ${computer.case1Price}<br>
        散热器: ${computer.cooler}<br>
        散热器价格: ${computer.coolerPrice}<br>
        显卡: ${computer.graphics_card}<br>
        显卡价格: ${computer.graphics_cardPrice}<br>
		硬盘: ${computer.hardDisk}<br>
		硬盘价格: ${computer.hardDiskPrice}<br>
		内存: ${computer.memory}<br>
		内存价格: ${computer.memoryPrice}<br>
		显示器: ${computer.monitor}<br>
		显示器价格: ${computer.monitorPrice}<br>
		电源: ${computer.powersupply}<br>
		电源价格: ${computer.powersupplyPrice}<br>
		总价: ${computer.totalPrice}<br>
        `;
      });
      // 更新Vue的数据，触发视图的更新
      this.$set(this.chatRecord, this.chatRecord.length - 1, ai);
    }
  }
}
</script>
