<template>
  <el-container>
    <div class="config-form">
      <h2>PalWorld INI配置文件生成器</h2>
      <el-table :data="tableData" stripe style="width: 100%">
        <el-table-column prop="key" label="字段" width="180"> </el-table-column>
        <el-table-column prop="value" label="内容" width="320">
          <template #default="scope">
            <input
              type="text"
              v-model="scope.row.value"
              class="ipt"
              style="width: 300px; text-align: center"
              @blur="iptBlur(scope.row, $event)"
            />
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
  </el-container>
</template>

<script setup>
import { ref } from "vue";
const tableData = ref([]);
const config = ref({});
const miaoshu = ref({
  AdminPassword: "服务器管理员密码",
  AutoResetGuildTimeNoOnlinePlayers: "默认：72(h)\n自动帮玩家退出公会的时间",
  bActiveUNKO: "默认：False\n神秘的UNKO事件关闭开启 (True/False)",
  BanListURL: "封锁名单 (目前来自官方)",
  BaseCampMaxNum: "基地最大数量",
  BaseCampWorkerMaxNum: "基地内工作帕鲁的最大数量",
  bAutoResetGuildNoOnlinePlayers:
    "默认：False\n自动帮玩家退出公会 (True/False)",
  bCanPickupOtherGuildDeathPenaltyDrop:
    "默认：False\n[推荐用于PVP] 是否捡取其他公会的死亡掉落物 (True/False)",
  bEnableAimAssistKeyboard: "默认：False\n键鼠辅助瞄准 (True/False)",
  bEnableAimAssistPad: "默认：False\n手把辅助瞄准 (True/False)",
  bEnableDefenseOtherGuildPlayer:
    "默认: False\n[推荐用于PVP] 是否受到其他公会伤害 (True/False)",
  bEnableFastTravel: "默认：True\n是否开启快速旅行 (True/False)",
  bEnableFriendlyFire:
    "默认: False\n[推荐用于PVP] 玩家对自己帕鲁跟同公会玩家的伤害 (True/False)",
  bEnableInvaderEnemy: "默认: True\n袭击事件关闭开启 (True/False)",
  bEnableNonLoginPenalty:
    "默认：False\n[推荐用于PVP] 登录惩罚 (True/False) [未知作用]",
  bEnablePlayerToPlayerDamage:
    "默认: False\n[推荐用于PVP] 玩家对玩家伤害 (True/False)",
  bExistPlayerAfterLogout: "默认: False\n是否玩家全部登出自动关服 (True/False)",
  bIsMultiplay: "默认: False\n未知作用 (True/False)",
  bIsPvP: "默认: False\n[推荐用于PVP] 多人游戏与PVP模式 (True/False)",
  bIsStartLocationSelectByMap:
    "默认: True\n是否创角后选择出生点 (True/False)\nPS: 若为关则出生在初始台地，死亡后依旧可以选择其他出生点",
  BuildObjectDamageRate: "默认：1\n建筑受伤害倍率 (0.5 - 3)",
  BuildObjectDeteriorationDamageRate: "默认：1\n建筑劣化速率 (0 - 10)",
  bUseAuth: "是否使用授权 (作用未知)/nPS: 应该是向官方验证的服务",
  CollectionDropRate:
    "默认：1\n采集资源倍率\n例 : 拿石稿敲小帕鲁矿取得1个帕鲁碎片默认要6下平均19点伤害，若调为3则要敲2下平均19点伤害。",
  CollectionObjectHpRate:
    "默认：1\n采集资源生命倍率 (0.5 - 3)\nPS: 经敲小帕鲁矿，测试不生效",
  CollectionObjectRespawnSpeedRate:
    "默认：1\n采集资源重生间隔 (0；0.5 - 3)\nPS: 树木大矿石为1小时，若为0立即重生",
  CoopPlayerMaxNum: "合作玩家人数",
  DayTimeSpeedRate: "默认：1\n白天速度 (0.1 - 5)",
  DeathPenalty:
    "死亡惩罚\nNone: 全部不掉落\nItem: 仅掉落道具(不含帕鲁及身上穿的装备武器)\nItemAndEquipment: 除了帕鲁其他全掉\nAll: 帕鲁、装备、道具全部掉落(不包含无法掉落的帕鲁专属装备，如马鞍)",
  Difficulty: "默认：None\n即使使用Difficult或是Easy也不影响下列调整的参数",
  DropItemAliveMaxHours:
    "默认：1(h)\n掉落物品的保留时间\nPS: 建议设为0.5以保证掉落物不会太多造成卡顿",
  DropItemMaxNum: "世界内的掉落物上限",
  DropItemMaxNum_UNKO: "神秘UNKO掉落物上限",
  EnemyDropItemRate: "默认：1\n敌人掉落倍率",
  ExpRate: "默认：1\n经验倍率",
  GuildPlayerMaxNum: "公会人数上限",
  NightTimeSpeedRate: "默认：1\n夜晚速度 (0.1 - 5)",
  PalAutoHPRegeneRate: "默认：1\n帕鲁的自动生命回复速率",
  PalAutoHpRegeneRateInSleep: "默认：1\n帕鲁睡眠中的生命回复倍率 (在帕鲁箱中)",
  PalCaptureRate:
    "默认：1\n帕鲁抓捕机率 (0.5 - 2)\nPS: 这个参数调整非常敏感请谨慎调整。困难模式为0.8，简单模式为2",
  PalDamageRateAttack: "默认：1\n帕鲁造成的伤害倍率",
  PalDamageRateDefense: "默认：1\n帕鲁受到的伤害倍率",
  PalEggDefaultHatchingTime: "默认：1(h)\n巨大蛋的孵化时间",
  PalSpawnNumRate:
    "默认：1\n帕鲁的重生数量\n若调整为2，王固定变2只，野生帕鲁群2~4只\nPS: 超过3会经常导致帕鲁重叠或掉出地图，请谨慎调整。",
  PalStaminaDecreaceRate: "默认：1\n帕鲁的耐力条下降速度 (0.1 - 5)",
  PalStomachDecreaceRate: "默认：1\n帕鲁的饱足感下降速度 (0.1 - 5)",
  PlayerAutoHPRegeneRate: "默认：1\n玩家的自动生命回复速率",
  PlayerAutoHpRegeneRateInSleep: "默认：1\n玩家睡眠中的生命回复倍率",
  PlayerDamageRateAttack: "默认：1\n玩家造成的伤害倍率 (0.1 - 5)",
  PlayerDamageRateDefense: "默认：1\n玩家受到的伤害倍率 (0.1 - 5)",
  PlayerStaminaDecreaceRate: "默认：1\n玩家的耐力条下降速度 (0.1 - 5)",
  PlayerStomachDecreaceRate: "默认：1\n玩家的饱足感下降速度 (0.1 - 5)",
  PublicIP: "主机位置",
  PublicPort: "主机端口",
  RCONEnabled: "RCON 使用或关闭",
  RCONPort: "RCON 端口",
  Region: "地区 (目前未知可能后期可以分国家)",
  ServerDescription: "服务器简介",
  ServerName: "服务器名称",
  ServerPassword: "服务器密码 (需等待官方更新耐久耐久修复)",
  ServerPlayerMaxNum: "服务器最大人数",
  WorkSpeedRate: "默认：1\n工作效率",
});
const iniContent = ref("");
// make table
const makeTable = () => {
  //循环miaoshu
  for (const [key, value] of Object.entries(miaoshu.value)) {
    tableData.value.push({
      key: key,
      value: config.value[key],
      label: value,
    });
  }
};
//失焦事件，使用事件对象拿到dom元素并进行后续样式的修改
const iptBlur = (value, e) => {
  // 输入框失焦之后，背景颜色变为粉色
  e.target.style.background = "pink";
};

// make form，根据miaoshu 生成表单
const makeForm = () => {
  config.value = {};
  // 动态生成表单
  let defaultConfig = `Difficulty=None,DayTimeSpeedRate=1.000000,NightTimeSpeedRate=1.000000,ExpRate=1.000000,PalCaptureRate=1.000000,PalSpawnNumRate=1.000000,PalDamageRateAttack=1.000000,PalDamageRateDefense=1.000000,PlayerDamageRateAttack=1.000000,PlayerDamageRateDefense=1.000000,PlayerStomachDecreaceRate=1.000000,PlayerStaminaDecreaceRate=1.000000,PlayerAutoHPRegeneRate=1.000000,PlayerAutoHpRegeneRateInSleep=1.000000,PalStomachDecreaceRate=1.000000,PalStaminaDecreaceRate=1.000000,PalAutoHPRegeneRate=1.000000,PalAutoHpRegeneRateInSleep=1.000000,BuildObjectDamageRate=1.000000,BuildObjectDeteriorationDamageRate=1.000000,CollectionDropRate=1.000000,CollectionObjectHpRate=1.000000,CollectionObjectRespawnSpeedRate=1.000000,EnemyDropItemRate=1.000000,DeathPenalty=All,bEnablePlayerToPlayerDamage=False,bEnableFriendlyFire=False,bEnableInvaderEnemy=True,bActiveUNKO=False,bEnableAimAssistPad=True,bEnableAimAssistKeyboard=False,DropItemMaxNum=3000,DropItemMaxNum_UNKO=100,BaseCampMaxNum=128,BaseCampWorkerMaxNum=15,DropItemAliveMaxHours=1.000000,bAutoResetGuildNoOnlinePlayers=False,AutoResetGuildTimeNoOnlinePlayers=72.000000,GuildPlayerMaxNum=20,PalEggDefaultHatchingTime=72.000000,WorkSpeedRate=1.000000,bIsMultiplay=False,bIsPvP=False,bCanPickupOtherGuildDeathPenaltyDrop=False,bEnableNonLoginPenalty=True,bEnableFastTravel=True,bIsStartLocationSelectByMap=True,bExistPlayerAfterLogout=False,bEnableDefenseOtherGuildPlayer=False,CoopPlayerMaxNum=4,ServerPlayerMaxNum=32,ServerName="CHIBANLI",ServerDescription="",AdminPassword="",ServerPassword="",PublicPort=8211,PublicIP="",RCONEnabled=False,RCONPort=25575,Region="",bUseAuth=True,BanListURL="https://api.palworldgame.com/api/banlist.txt"`;
  // 解析defaultConfig,逗号分割，等于号区分
  let defaultConfigList = defaultConfig.split(",");
  for (let i = 0; i < defaultConfigList.length; i++) {
    let item = defaultConfigList[i].split("=");
    config.value[item[0]] = item[1];
  }
  makeTable();
};
const generateIni = () => {
  let header = `[/Script/Pal.PalGameWorldSettings]
OptionSettings=(`;
  let footer = `)`;
  let tmp = "";
  for (const [key, value] of Object.entries(config.value)) {
    tmp += `${key}=${value}` + ",";
  }
  tmp = tmp.slice(0, -1);
  // 去除换行符
  tmp =
    header +
    tmp +
    footer.replace(/\n/g, "").replace(/\r/g, "").replace(/\t/g, "");
  iniContent.value = tmp;
};
makeForm();
setInterval(() => {
  generateIni();
}, 100);
</script>

<style scoped>
/* 这里是一些样式 */
</style>
