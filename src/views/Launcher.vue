<template>
  <el-row :gutter="16">
    <el-col :span="6">
      <el-card style="height: 789px;">
        <img src="../assets/world-of-warcraft.png" width="100%" />
        <div style="margin-top: 16px;line-height: 12px;font-size: 12px;">
          <el-row :gutter="16">
            <el-col :span="16">
              Mysql:
              <span v-if="process.mysql.length > 0">
                [{{ process.mysql.toString() }}]
              </span>
            </el-col>
            <el-col :span="8" style="text-align:right;">
              <i
                class="el-icon-loading"
                style="margin-right: 4px"
                v-if="disabled.mysql"
              />
              <el-switch
                v-model="status.mysql"
                :disabled="disabled.mysql"
                @change="(status) => handleChange('mysql', status)"
              >
              </el-switch>
            </el-col>
          </el-row>
          <el-row :gutter="16" style="margin-top: 8px;">
            <el-col :span="16">
              World Server:
              <span v-if="process.worldServer.length > 0">
                [{{ process.worldServer.toString() }}]
              </span>
            </el-col>
            <el-col :span="8" style="text-align:right;">
              <i
                class="el-icon-loading"
                style="margin-right: 4px"
                v-if="disabled.worldServer"
              />
              <el-switch
                v-model="status.worldServer"
                :disabled="disabled.worldServer"
                @change="(status) => handleChange('worldServer', status)"
              >
              </el-switch>
            </el-col>
          </el-row>
          <el-row :gutter="16" style="margin-top: 8px;">
            <el-col :span="16">
              Auth Server:
              <span v-if="process.authServer.length > 0">
                [{{ process.authServer.toString() }}]
              </span>
            </el-col>
            <el-col :span="8" style="text-align:right;">
              <i
                class="el-icon-loading"
                style="margin-right: 4px"
                v-if="disabled.authServer"
              />
              <el-switch
                v-model="status.authServer"
                :disabled="disabled.authServer"
                @change="(status) => handleChange('authServer', status)"
              >
              </el-switch>
            </el-col>
          </el-row>
        </div>
        <div style="position: absolute; bottom: 16px;">
          <div style="position: relative;">
            <div style="height: 12px;line-height: 12px;font-size: 12px;">
              游戏版本
            </div>
            <el-select v-model="version" style="margin-top: 8px;width: 260px;">
              <el-option label="AzerothCore" value="AzerothCore"></el-option>
              <el-option label="FoxCore" value="FoxCore"></el-option>
            </el-select>
            <div style="margin-top: 8px;">
              <el-button
                type="primary"
                style="border-radius: 4px 0 0 4px;width:228px;"
                @click="enterGame"
              >
                进入游戏
              </el-button>

              <el-dropdown
                type="primary"
                placement="top-start"
                trigger="click"
                @command="handleCommand"
              >
                <el-button
                  type="primary"
                  icon="el-icon-setting"
                  style="margin-left: 0px;padding: 12px 8px;border-radius: 0 4px 4px 0;border-left-color: #ffffff;"
                ></el-button>
                <el-dropdown-menu slot="dropdown">
                  <el-dropdown-item command="start-all">
                    启动所有服务
                  </el-dropdown-item>
                  <el-dropdown-item command="stop-all">
                    停止所有服务
                  </el-dropdown-item>
                  <el-dropdown-item command="start-client" divided>
                    启动客户端
                  </el-dropdown-item>
                </el-dropdown-menu>
              </el-dropdown>
            </div>
          </div>
        </div>
      </el-card>
    </el-col>
    <el-col :span="18" style="font-size: 10px;">
      <el-card
        ref="mysql"
        style="height: 251px; margin: 0 0 0 16px;overflow: auto;"
      >
        <div v-html="console.mysql.replaceAll('\n', '<br>')"></div>
      </el-card>
      <el-card
        ref="worldServer"
        style="height: 251px; margin: 16px 0 0 16px;overflow: auto;"
      >
        <div v-html="console.worldServer.replaceAll('\n', '<br>')"></div>
      </el-card>
      <el-card
        ref="authServer"
        style="height: 251px; margin: 16px 0 0 16px;overflow: auto;"
      >
        <div v-html="console.authServer.replaceAll('\n', '<br>')"></div>
      </el-card>
    </el-col>
  </el-row>
</template>

<script>
const ipcRenderer = window.ipcRenderer;
import { mapState, mapActions } from "vuex";

export default {
  data() {
    return {
      version: "AzerothCore",
    };
  },
  computed: {
    ...mapState("launcher", ["status", "disabled", "process", "console"]),
  },
  methods: {
    ...mapActions("launcher", ["updateStatus"]),
    handleChange(service, status) {
      this.updateStatus({ service, status });
      ipcRenderer.send("MANAGE_SERVICE", { service, status });
    },
    enterGame() {
      this.updateStatus({ service: "mysql", status: true });
      this.updateStatus({ service: "worldServer", status: true });
      this.updateStatus({ service: "authServer", status: true });
      ipcRenderer.send("ENTER_GAME");
    },
    handleCommand(command) {
      switch (command) {
        case "start-all":
          this.updateStatus({ service: "mysql", status: true });
          this.updateStatus({ service: "worldServer", status: true });
          this.updateStatus({ service: "authServer", status: true });
          ipcRenderer.send("START_ALL");
          break;
        case "stop-all":
          this.updateStatus({ service: "mysql", status: false });
          this.updateStatus({ service: "worldServer", status: false });
          this.updateStatus({ service: "authServer", status: false });
          ipcRenderer.send("STOP_ALL");
          break;
        case "start-client":
          ipcRenderer.send("START_CLIENT");
          break;
        default:
          break;
      }
    },
  },
  mounted() {
    this.$nextTick(() => {
      let element = this.$refs["mysql"].$el;
      element.scrollTop = element.scrollHeight;
      element = this.$refs["worldServer"].$el;
      element.scrollTop = element.scrollHeight;
      element = this.$refs["authServer"].$el;
      element.scrollTop = element.scrollHeight;
    });
    ipcRenderer.on("CHILD_PROCESS_STDOUT", (event, payload) => {
      this.$nextTick(() => {
        let element = this.$refs[payload.channel].$el;
        element.scrollTop = element.scrollHeight;
      });
    });
  },
};
</script>
