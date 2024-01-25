<template>
  <el-container>
    <div class="config-form">
      <h2>PalWorld INI配置文件生成器</h2>
      <el-row class="row-bg" justify="space-evenly">
        <el-col :span="6">
          <el-upload
            ref="uploadRef"
            class="upload-demo"
            :show-file-list="false"
            :before-upload="uploadIni"
          >
            <template #trigger>
              <el-button type="primary">上传文件</el-button>
            </template>
          </el-upload>
        </el-col>

        <el-col :span="6">
          <el-button type="primary" @click="downloadIni"
            >下载配置文件</el-button
          >
        </el-col>
        <el-col :span="6">
          <el-button type="primary" @click="inputData">导入字符串</el-button>
        </el-col>
        <el-col :span="6">
          <el-button type="primary" @click="showData">导出字符串</el-button>
        </el-col>
      </el-row>
      <el-table :data="tableData" stripe style="width: 100%">
        <el-table-column prop="key" label="字段" width="180"> </el-table-column>
        <el-table-column prop="value" label="内容" width="320">
          <template #default="scope">
            <el-input
              type="text"
              v-model="scope.row.value"
              class="ipt"
              @blur="iptBlur(scope.row, $event)"
              v-if="scope.row.xtype == 'string' || scope.row.xtype == 'none'"
            />
            <el-switch
              v-model="scope.row.value"
              active-color="#13ce66"
              inactive-color="#ff4949"
              v-if="scope.row.xtype == 'bool'"
            />
            <el-select
              v-model="scope.row.value"
              v-if="scope.row.xtype.indexOf('|') >= 0"
            >
              <el-option
                v-for="item in scope.row.xtype.split('|')"
                :key="item"
                :label="item"
                :value="item"
              />
            </el-select>
          </template>
        </el-table-column>
        <el-table-column prop="label" label="描述" />
      </el-table>
      <!-- 显示生成的INI配置 -->
      <el-input
        v-model="iniContent"
        :autosize="{ minRows: 20, maxRows: 40 }"
        type="textarea"
        placeholder="配置文件"
        style="margin-top: 20px"
      />
    </div>
  <el-backtop :right="100" :bottom="100" />
  </el-container>
</template>

<script setup>
import {ref ,watch} from "vue";
const tableData =ref([]);
const config = ref({});
const miaoshu = ref([
  { name: "AdminPassword", label: "服务器管理员密码", type: "string" },
  {
    name: "AutoResetGuildTimeNoOnlinePlayers",
    label: "默认：72(h)\n自动帮玩家退出公会的时间",
    type: "none",
  },
  {
    name: "bActiveUNKO",
    label: "默认：False\n神秘的UNKO事件关闭开启 (True/False)",
    type: "bool",
  },
  { name: "BanListURL", label: "封锁名单 (目前来自官方)", type: "string" },
  { name: "BaseCampMaxNum", label: "基地最大数量", type: "none" },
  {
    name: "BaseCampWorkerMaxNum",
    label: "基地内工作帕鲁的最大数量",
    type: "none",
  },
  {
    name: "bAutoResetGuildNoOnlinePlayers",
    label: "默认：False\n自动帮玩家退出公会 (True/False)",
    type: "bool",
  },
  {
    name: "bCanPickupOtherGuildDeathPenaltyDrop",
    label:
      "默认：False\n[推荐用于PVP] 是否捡取其他公会的死亡掉落物 (True/False)",
    type: "bool",
  },
  {
    name: "bEnableAimAssistKeyboard",
    label: "默认：False\n键鼠辅助瞄准 (True/False)",
    type: "bool",
  },
  {
    name: "bEnableAimAssistPad",
    label: "默认：False\n手把辅助瞄准 (True/False)",
    type: "bool",
  },
  {
    name: "bEnableDefenseOtherGuildPlayer",
    label: "默认: False\n[推荐用于PVP] 是否受到其他公会伤害 (True/False)",
    type: "bool",
  },
  {
    name: "bEnableFastTravel",
    label: "默认：True\n是否开启快速旅行 (True/False)",
    type: "bool",
  },
  {
    name: "bEnableFriendlyFire",
    label:
      "默认: False\n[推荐用于PVP] 玩家对自己帕鲁跟同公会玩家的伤害 (True/False)",
    type: "bool",
  },
  {
    name: "bEnableInvaderEnemy",
    label: "默认: True\n袭击事件关闭开启 (True/False)",
    type: "bool",
  },
  {
    name: "bEnableNonLoginPenalty",
    label: "默认：False\n[推荐用于PVP] 登录惩罚 (True/False) [未知作用]",
    type: "bool",
  },
  {
    name: "bEnablePlayerToPlayerDamage",
    label: "默认: False\n[推荐用于PVP] 玩家对玩家伤害 (True/False)",
    type: "bool",
  },
  {
    name: "bExistPlayerAfterLogout",
    label: "默认: False\n 玩家登出是否留在原地（不消失） (True/False)",
    type: "bool",
  },
  {
    name: "bIsMultiplay",
    label: "默认: False\n未知作用 (True/False)",
    type: "bool",
  },
  {
    name: "bIsPvP",
    label: "默认: False\n[推荐用于PVP] 多人游戏与PVP模式 (True/False)",
    type: "bool",
  },
  {
    name: "bIsStartLocationSelectByMap",
    label:
      "默认: True\n是否创角后选择出生点 (True/False)\nPS: 若为关则出生在初始台地，死亡后依旧可以选择其他出生点",
    type: "bool",
  },
  {
    name: "BuildObjectDamageRate",
    label: "默认：1\n建筑受伤害倍率 (0.5 - 3)",
    type: "none",
  },
  {
    name: "BuildObjectDeteriorationDamageRate",
    label: "默认：1\n建筑劣化速率 (0 - 10)",
    type: "none",
  },
  {
    name: "bUseAuth",
    label: "是否使用授权 (作用未知)/nPS: 应该是向官方验证的服务",
    type: "bool",
  },
  {
    name: "CollectionDropRate",
    label:
      "默认：1\n采集资源倍率\n例 : 拿石稿敲小帕鲁矿取得1个帕鲁碎片默认要6下平均19点伤害，若调为3则要敲2下平均19点伤害。",
    type: "none",
  },
  {
    name: "CollectionObjectHpRate",
    label: "默认：1\n采集资源生命倍率 (0.5 - 3)\nPS: 经敲小帕鲁矿，测试不生效",
    type: "none",
  },
  {
    name: "CollectionObjectRespawnSpeedRate",
    label:
      "默认：1\n采集资源重生间隔 (0；0.5 - 3)\nPS: 树木大矿石为1小时，若为0立即重生",
    type: "none",
  },
  { name: "CoopPlayerMaxNum", label: "合作玩家人数", type: "none" },
  {
    name: "DayTimeSpeedRate",
    label: "默认：1\n白天速度 (0.1 - 5)",
    type: "none",
  },
  {
    name: "DeathPenalty",
    label:
      "死亡惩罚\nNone: 全部不掉落\nItem: 仅掉落道具(不含帕鲁及身上穿的装备武器)\nItemAndEquipment: 除了帕鲁其他全掉\nAll: 帕鲁、装备、道具全部掉落(不包含无法掉落的帕鲁专属装备，如马鞍)",
    type: "None|Item|ItemAndEquipment|All",
  },
  {
    name: "Difficulty",
    label: "默认：None\n即使使用Difficult或是Easy也不影响下列调整的参数",
    type: "None|Easy|Normal|Hard|1|2|3",
  },
  {
    name: "DropItemAliveMaxHours",
    label:
      "默认：1(h)\n掉落物品的保留时间\nPS: 建议设为0.5以保证掉落物不会太多造成卡顿",
    type: "none",
  },
  { name: "DropItemMaxNum", label: "世界内的掉落物上限", type: "none" },
  { name: "DropItemMaxNum_UNKO", label: "神秘UNKO掉落物上限", type: "none" },
  { name: "EnemyDropItemRate", label: "默认：1\n敌人掉落倍率", type: "none" },
  { name: "ExpRate", label: "默认：1\n经验倍率", type: "none" },
  { name: "GuildPlayerMaxNum", label: "公会人数上限", type: "none" },
  {
    name: "NightTimeSpeedRate",
    label: "默认：1\n夜晚速度 (0.1 - 5)",
    type: "none",
  },
  {
    name: "PalAutoHPRegeneRate",
    label: "默认：1\n帕鲁的自动生命回复速率",
    type: "none",
  },
  {
    name: "PalAutoHpRegeneRateInSleep",
    label: "默认：1\n帕鲁睡眠中的生命回复倍率 (在帕鲁箱中)",
    type: "none",
  },
  {
    name: "PalCaptureRate",
    label:
      "默认：1\n帕鲁抓捕机率 (0.5 - 2)\nPS: 这个参数调整非常敏感请谨慎调整。困难模式为0.8，简单模式为2",
    type: "none",
  },
  {
    name: "PalDamageRateAttack",
    label: "默认：1\n帕鲁造成的伤害倍率",
    type: "none",
  },
  {
    name: "PalDamageRateDefense",
    label: "默认：1\n帕鲁受到的伤害倍率",
    type: "none",
  },
  {
    name: "PalEggDefaultHatchingTime",
    label: "默认：1(h)\n巨大蛋的孵化时间",
    type: "none",
  },
  {
    name: "PalSpawnNumRate",
    label:
      "默认：1\n帕鲁的重生数量\n若调整为2，王固定变2只，野生帕鲁群2~4只\nPS: 超过3会经常导致帕鲁重叠或掉出地图，请谨慎调整。",
    type: "none",
  },
  {
    name: "PalStaminaDecreaceRate",
    label: "默认：1\n帕鲁的耐力条下降速度 (0.1 - 5)",
    type: "none",
  },
  {
    name: "PalStomachDecreaceRate",
    label: "默认：1\n帕鲁的饱足感下降速度 (0.1 - 5)",
    type: "none",
  },
  {
    name: "PlayerAutoHPRegeneRate",
    label: "默认：1\n玩家的自动生命回复速率",
    type: "none",
  },
  {
    name: "PlayerAutoHpRegeneRateInSleep",
    label: "默认：1\n玩家睡眠中的生命回复倍率",
    type: "none",
  },
  {
    name: "PlayerDamageRateAttack",
    label: "默认：1\n玩家造成的伤害倍率 (0.1 - 5)",
    type: "none",
  },
  {
    name: "PlayerDamageRateDefense",
    label: "默认：1\n玩家受到的伤害倍率 (0.1 - 5)",
    type: "none",
  },
  {
    name: "PlayerStaminaDecreaceRate",
    label: "默认：1\n玩家的耐力条下降速度 (0.1 - 5)",
    type: "none",
  },
  {
    name: "PlayerStomachDecreaceRate",
    label: "默认：1\n玩家的饱足感下降速度 (0.1 - 5)",
    type: "none",
  },
  { name: "PublicIP", label: "主机位置", type: "string" },
  { name: "PublicPort", label: "主机端口", type: "none" },
  { name: "RCONEnabled", label: "RCON 使用或关闭", type: "bool" },
  { name: "RCONPort", label: "RCON 端口", type: "none" },
  {
    name: "Region",
    label: "地区 (目前未知可能后期可以分国家)",
    type: "string",
  },
  { name: "ServerDescription", label: "服务器简介", type: "string" },
  { name: "ServerName", label: "服务器名称", type: "string" },
  {
    name: "ServerPassword",
    label: "服务器密码 (需等待官方更新耐久耐久修复)",
    type: "string",
  },
  { name: "ServerPlayerMaxNum", label: "服务器最大人数", type: "none" },
  { name: "WorkSpeedRate", label: "默认：1\n工作效率", type: "none" },
]);
const iniContent = ref("");
// make table
const makeTable = () => {
  // tableData 清空
  tableData.value = [];
  // 按照config的顺序生成表格
  for (let k in config.value) {
    //找到对应的描述
    let itemDesc = miaoshu.value.find((item) => item.name == k);
    if (itemDesc) {
      let xvalue = "";
      if (itemDesc.type == "bool") {
        if (config.value[k] == "False") {
          xvalue = false;
        } else {
          xvalue = true;
        }
      } else if (itemDesc.type == "string") {
        //去除双引号
        xvalue = config.value[k].replace(/"/g, "");
      } else {
        xvalue = config.value[k];
      }
      tableData.value.push({
        key: itemDesc.name,
        value: xvalue,
        label: itemDesc.label,
        xtype: itemDesc.type,
      });
    }
  }
};
//失焦事件，使用事件对象拿到dom元素并进行后续样式的修改
const iptBlur = (value, e) => {
  // 输入框失焦之后，背景颜色变为粉色
  e.target.style.background = "pink";
};

// make form，根据miaoshu 生成表单
const makeForm = (defaultConfig = "") => {
  config.value = {};
  if (defaultConfig == "") {
    // 动态生成表单
    defaultConfig = `Difficulty=None,DayTimeSpeedRate=1.000000,NightTimeSpeedRate=1.000000,ExpRate=1.000000,PalCaptureRate=1.000000,PalSpawnNumRate=1.000000,PalDamageRateAttack=1.000000,PalDamageRateDefense=1.000000,PlayerDamageRateAttack=1.000000,PlayerDamageRateDefense=1.000000,PlayerStomachDecreaceRate=1.000000,PlayerStaminaDecreaceRate=1.000000,PlayerAutoHPRegeneRate=1.000000,PlayerAutoHpRegeneRateInSleep=1.000000,PalStomachDecreaceRate=1.000000,PalStaminaDecreaceRate=1.000000,PalAutoHPRegeneRate=1.000000,PalAutoHpRegeneRateInSleep=1.000000,BuildObjectDamageRate=1.000000,BuildObjectDeteriorationDamageRate=1.000000,CollectionDropRate=1.000000,CollectionObjectHpRate=1.000000,CollectionObjectRespawnSpeedRate=1.000000,EnemyDropItemRate=1.000000,DeathPenalty=All,bEnablePlayerToPlayerDamage=False,bEnableFriendlyFire=False,bEnableInvaderEnemy=True,bActiveUNKO=False,bEnableAimAssistPad=True,bEnableAimAssistKeyboard=False,DropItemMaxNum=3000,DropItemMaxNum_UNKO=100,BaseCampMaxNum=128,BaseCampWorkerMaxNum=15,DropItemAliveMaxHours=1.000000,bAutoResetGuildNoOnlinePlayers=False,AutoResetGuildTimeNoOnlinePlayers=72.000000,GuildPlayerMaxNum=20,PalEggDefaultHatchingTime=72.000000,WorkSpeedRate=1.000000,bIsMultiplay=False,bIsPvP=False,bCanPickupOtherGuildDeathPenaltyDrop=False,bEnableNonLoginPenalty=True,bEnableFastTravel=True,bIsStartLocationSelectByMap=True,bExistPlayerAfterLogout=False,bEnableDefenseOtherGuildPlayer=False,CoopPlayerMaxNum=4,ServerPlayerMaxNum=32,ServerName="CHIBANLI",ServerDescription="",AdminPassword="",ServerPassword="",PublicPort=8211,PublicIP="",RCONEnabled=False,RCONPort=25575,Region="",bUseAuth=True,BanListURL="https://api.palworldgame.com/api/banlist.txt"`;
  } else {
    // 使用"(" 分割字符串
    let defaultConfigList = defaultConfig.split("(");
    defaultConfig = defaultConfigList[1];
    // 使用 ")" 分割字符串
    defaultConfigList = defaultConfig.split(")");
    defaultConfig = defaultConfigList[0];
  }
  // 解析defaultConfig,逗号分割，等于号区分
  let defaultConfigList = defaultConfig.split(",");
  for (let i = 0; i < defaultConfigList.length; i++) {
    let item = defaultConfigList[i].split("=");
    config.value[item[0]] = item[1];
  }
  makeTable();
};
// 导入字符串
const inputData = () => {
  // 弹窗让用户输入
  ElMessageBox.prompt("请输入字符串", "导入字符串", {
    confirmButtonText: "确定",
    cancelButtonText: "取消",
    inputPattern: /\S/,
    inputErrorMessage: "字符串不能为空",
    inputType: "textarea",
  })
    .then(({ value }) => {
      makeForm(value);
    })
    .catch(() => {});
};
const uploadRef = ref();
//导入文件
const uploadIni = (file) => {
  console.log(file)
  //beforeUpload
  //读取文件
  let reader = new FileReader();
  reader.onload = (e) => {
    console.log(e)
    makeForm(e.target.result);
  };

  reader.readAsText(file);
};
//下载文件
const downloadIni = () => {
  // 获取iniContent.value 并生成文件
  let blob = new Blob([iniContent.value], { type: "text/plain;charset=utf-8" });
  let downloadElement = document.createElement("a");
  let href = window.URL.createObjectURL(blob); //创建下载的链接
  downloadElement.href = href;
  downloadElement.download = "PalWorldSettings.ini"; //下载后文件名
  document.body.appendChild(downloadElement);
  downloadElement.click(); //点击下载
  document.body.removeChild(downloadElement); //下载完成移除元素
  window.URL.revokeObjectURL(href); //释放掉blob对象
  ElMessage({
    message: "下载成功",
    type: "success",
  });
};
const showData = () => {
  //滚动整个页面到底部
  window.scrollTo(0, document.body.scrollHeight);
  //复制到剪贴板
  navigator.clipboard.writeText(iniContent.value);
  ElMessage({
    type: "success",
    message: "复制成功",
  })
}
const generateIni = () => {
  let header = `[/Script/Pal.PalGameWorldSettings]
OptionSettings=(`;
  let footer = `)`;
  let tmp = "";
  // 按照config的顺序生成ini
  for (let i = 0; i < tableData.value.length; i++) {
    if (tableData.value[i].xtype == "bool") {
      if (tableData.value[i].value) {
        tmp += `${tableData.value[i].key}=True,`;
      } else {
        tmp += `${tableData.value[i].key}=False,`;
      }
    } else if (tableData.value[i].xtype == "string") {
      tmp += `${tableData.value[i].key}="${tableData.value[i].value}",`;
    } else {
      tmp += `${tableData.value[i].key}=${tableData.value[i].value},`;
    }
  }
  tmp = tmp.slice(0, -1);
  // 去除换行符
  tmp =
    header +
    tmp +
    footer.replace(/\n/g, "").replace(/\r/g, "").replace(/\t/g, "");
  iniContent.value = tmp;
};
//监听键盘事件，ctrl+s 保存
document.onkeydown = function (e) {
  if (e.ctrlKey && e.key == "s") {
    e.preventDefault();
    downloadIni();
  }
}
makeForm();

//watch tabledata.value
watch(tableData, (newVal, oldVal) => {
  generateIni();
},{deep:true,immediate:true});

</script>

<style scoped>
/* 这里是一些样式 */
</style>
