<template>
  <el-container>
      <div class="config-form">
        <h2>PalWorld INI配置文件生成器</h2>
        <el-table :data="tableData" stripe style="width: 100%">
          <el-table-column prop="key" label="字段" width="180">
          </el-table-column>
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
          style="margin-top: 20px;"
        />
      </div>
  </el-container>
</template>

<script setup>
import { ref } from "vue";
const tableData = ref([]);
const config = ref({});
const miaoshu = ref({
  AdminPassword: "伺服器管理員密碼",
  AutoResetGuildTimeNoOnlinePlayers: "預設：72(h)\n自動幫玩家退出公會的時間",
  bActiveUNKO: "預設：False\n謎之UNKO事件關閉開啟 (True/False)",
  BanListURL: "封鎖名單 (目前來自官方)",
  BaseCampMaxNum: "基地最大數量",
  BaseCampWorkerMaxNum: "基地內工作帕魯的最大數量",
  bAutoResetGuildNoOnlinePlayers:
    "預設：False\n自動幫玩家退出公會 (True/False)",
  bCanPickupOtherGuildDeathPenaltyDrop:
    "預設：False\n[推薦用於PVP] 是否撿取其他公會的死亡掉落物 (True/False)",
  bEnableAimAssistKeyboard: "預設：False\n鍵鼠輔助瞄準 (True/False)",
  bEnableAimAssistPad: "預設：False\n手把輔助瞄準 (True/False)",
  bEnableDefenseOtherGuildPlayer:
    "預設: False\n[推薦用於PVP] 是否受到其他公會傷害 (True/False)",
  bEnableFastTravel: "預設：True\n是否開啟快速旅行 (True/False)",
  bEnableFriendlyFire:
    "預設: False\n[推薦用於PVP] 玩家對自己帕魯跟同公會玩家的傷害 (True/False)",
  bEnableInvaderEnemy: "預設: True\n襲擊事件關閉開啟 (True/False)",
  bEnableNonLoginPenalty:
    "預設：False\n[推薦用於PVP] 登入懲罰 (True/False) [未知作用]",
  bEnablePlayerToPlayerDamage:
    "預設: False\n[推薦用於PVP] 玩家對玩家傷害 (True/False)",
  bExistPlayerAfterLogout: "預設: False\n是否玩家全部登出自動關服 (True/False)",
  bIsMultiplay: "預設: False\n未知作用 (True/False)",
  bIsPvP: "預設: False\n[推薦用於PVP] 多人遊戲與PVP模式 (True/False)",
  bIsStartLocationSelectByMap:
    "預設: True\n是否創角後選擇出生點 (True/False)\nPS: 若為關則出生在初始台地，死亡後依舊可以選擇其他出生點",
  BuildObjectDamageRate: "預設：1\n建築受傷害倍率 (0.5 - 3)",
  BuildObjectDeteriorationDamageRate: "預設：1\n建築劣化速率 (0 - 10)",
  bUseAuth: "是否使用授權 (作用未知)/nPS: 應該是向官方驗證的服務",
  CollectionDropRate:
    "預設：1\n採集資源被率\n例 : 拿石稿敲小帕魯礦取得1個帕魯碎片預設要6下平均19點傷害，若調為3則要敲2下平均19點傷害。",
  CollectionObjectHpRate:
    "預設：1\n採集資源生命倍率 (0.5 - 3)\nPS: 經敲小帕魯礦，測試不生效",
  CollectionObjectRespawnSpeedRate:
    "預設：1\n採集資源重生間格 (0；0.5 - 3)\nPS: 樹木大礦石為1小時，若為0立即重生",
  CoopPlayerMaxNum: "合作玩家人數",
  DayTimeSpeedRate: "預設：1\n白天速度 (0.1 - 5)",
  DeathPenalty:
    "死亡懲罰\nNone: 全部不掉落\nItem: 僅掉落道具(不含帕魯及身上穿的裝備武器)\nItemAndEquipment: 除了帕魯其他全掉\nAll: 帕魯、裝備、道具全部掉落(不包含無法掉落的帕魯專屬裝備，如馬鞍)",
  Difficulty: "預設：None\n即使使用Difficult或是Easy也不影響下列調整的參數",
  DropItemAliveMaxHours:
    "預設：1(h)\n掉落物品的保留時間\nPS: 建議設為0.5以保證掉落物不會太多造成卡頓",
  DropItemMaxNum: "世界內的掉落物上限",
  DropItemMaxNum_UNKO: "謎之UNKO掉落物上限",
  EnemyDropItemRate: "預設：1\n敵人掉落倍率",
  ExpRate: "預設：1\n經驗倍率",
  GuildPlayerMaxNum: "公會人數上限",
  NightTimeSpeedRate: "預設：1\n夜晚速度 (0.1 - 5)",
  PalAutoHPRegeneRate: "預設：1\n帕魯的自動生命回復速率",
  PalAutoHpRegeneRateInSleep: "預設：1\n帕魯睡眠中的生命回復倍率 (在帕魯箱中)",
  PalCaptureRate:
    "預設：1\n帕魯抓捕機率 (0.5 - 2)\nPS: 這個參數調整非常敏感請謹慎調整。困難模式為0.8，簡單模式為2",
  PalDamageRateAttack: "預設：1\n帕魯造成的傷害倍率",
  PalDamageRateDefense: "預設：1\n帕魯受到的傷害倍率",
  PalEggDefaultHatchingTime: "預設：1(h)\n巨大蛋的孵化時間",
  PalSpawnNumRate:
    "預設：1\n帕魯的重生數量\n若調整為2，王固定變2隻，野生帕魯群2~4隻\nPS: 超過3會經常導致帕魯重疊或掉出地圖，請謹慎調整。",
  PalStaminaDecreaceRate: "預設：1\n帕魯的耐力條下降速度 (0.1 - 5)",
  PalStomachDecreaceRate: "預設：1\n帕魯的飽足感下降速度 (0.1 - 5)",
  PlayerAutoHPRegeneRate: "預設：1\n玩家的自動生命回復速率",
  PlayerAutoHpRegeneRateInSleep: "預設：1\n玩家睡眠中的生命回復倍率",
  PlayerDamageRateAttack: "預設：1\n玩家造成的傷害倍率 (0.1 - 5)",
  PlayerDamageRateDefense: "預設：1\n玩家受到的傷害倍率 (0.1 - 5)",
  PlayerStaminaDecreaceRate: "預設：1\n玩家的耐力條下降速度 (0.1 - 5)",
  PlayerStomachDecreaceRate: "預設：1\n玩家的飽足感下降速度 (0.1 - 5)",
  PublicIP: "主機位置",
  PublicPort: "主機端口",
  RCONEnabled: "RCON 使用或關閉",
  RCONPort: "RCON 端口",
  Region: "地區 (目前未知可能後期可以分國家)",
  ServerDescription: "伺服器簡介",
  ServerName: "伺服器名稱",
  ServerPassword: "伺服器密碼 (需等待官方更新耐久耐久修復)",
  ServerPlayerMaxNum: "伺服器最大人數",
  WorkSpeedRate: "預設：1\n工作效率",
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
