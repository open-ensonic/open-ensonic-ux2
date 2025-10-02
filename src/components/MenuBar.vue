<template>
  <div class="menu-bar">
    <el-menu
      mode="horizontal"
      :ellipsis="false"
      background-color="#545c64"
      text-color="#fff"
      active-text-color="#ffd04b"
      class="main-menu"
    >
      <!-- 文件菜单 -->
      <el-sub-menu index="file">
        <template #title>文件</template>
        <el-sub-menu index="file-new">
          <template #title>新建</template>
          <el-menu-item index="file-new-qt" @click="emit('menu-action', 'file.new', { type: 'qt' })">
            QT GUI
          </el-menu-item>
          <el-menu-item index="file-new-no-gui" @click="emit('menu-action', 'file.new', { type: 'no-gui' })">
            NO GUI
          </el-menu-item>
          <el-menu-item index="file-new-hier" @click="emit('menu-action', 'file.new', { type: 'hier' })">
            Hier Block
          </el-menu-item>
          <el-menu-item index="file-new-hier-qt" @click="emit('menu-action', 'file.new', { type: 'hier-qt' })">
            Hier Block(QT GUI)
          </el-menu-item>
        </el-sub-menu>
        <el-menu-item index="file-duplicate" @click="emit('menu-action', 'file.duplicate')">
          复制
        </el-menu-item>
        <el-divider />
        <el-menu-item index="file-open" @click="emit('menu-action', 'file.open')">
          <el-icon><FolderOpened /></el-icon>
          打开
        </el-menu-item>
        <el-sub-menu index="file-recent">
          <template #title>打开最近</template>
          <el-menu-item index="recent-1">项目1.grc</el-menu-item>
          <el-menu-item index="recent-2">项目2.grc</el-menu-item>
        </el-sub-menu>
        <el-divider />
        <el-menu-item index="file-save" @click="emit('menu-action', 'file.save')">
          <el-icon><Document /></el-icon>
          保存
        </el-menu-item>
        <el-menu-item index="file-save-as" @click="emit('menu-action', 'file.saveAs')">
          另存为
        </el-menu-item>
        <el-divider />
        <el-menu-item index="file-screenshot" @click="emit('menu-action', 'file.screenshot')">
          <el-icon><Camera /></el-icon>
          截图
        </el-menu-item>
        <el-divider />
        <el-menu-item index="file-exit" @click="emit('menu-action', 'file.exit')">
          退出
        </el-menu-item>
      </el-sub-menu>

      <!-- 编辑菜单 -->
      <el-sub-menu index="edit">
        <template #title>编辑</template>
        <el-menu-item index="edit-undo" @click="emit('menu-action', 'edit.undo')">
          <el-icon><RefreshLeft /></el-icon>
          撤销
        </el-menu-item>
        <el-menu-item index="edit-redo" @click="emit('menu-action', 'edit.redo')">
          <el-icon><RefreshRight /></el-icon>
          重做
        </el-menu-item>
        <el-divider />
        <el-menu-item index="edit-cut" @click="emit('menu-action', 'edit.cut')">
          <el-icon><CopyDocument /></el-icon>
          剪切
        </el-menu-item>
        <el-menu-item index="edit-copy" @click="emit('menu-action', 'edit.copy')">
          <el-icon><CopyDocument /></el-icon>
          复制
        </el-menu-item>
        <el-menu-item index="edit-paste" @click="emit('menu-action', 'edit.paste')">
          <el-icon><DocumentCopy /></el-icon>
          粘贴
        </el-menu-item>
        <el-menu-item index="edit-delete" @click="emit('menu-action', 'edit.delete')">
          <el-icon><Delete /></el-icon>
          删除
        </el-menu-item>
        <el-menu-item index="edit-select-all" @click="emit('menu-action', 'edit.selectAll')">
          全选
        </el-menu-item>
        <el-divider />
        <el-sub-menu index="edit-align">
          <template #title>对齐</template>
          <el-menu-item index="align-top" @click="emit('menu-action', 'edit.align', { type: 'top' })">
            顶端对齐
          </el-menu-item>
          <el-menu-item index="align-middle" @click="emit('menu-action', 'edit.align', { type: 'middle' })">
            中部对齐
          </el-menu-item>
          <el-menu-item index="align-bottom" @click="emit('menu-action', 'edit.align', { type: 'bottom' })">
            底端对齐
          </el-menu-item>
          <el-menu-item index="align-left" @click="emit('menu-action', 'edit.align', { type: 'left' })">
            左对齐
          </el-menu-item>
          <el-menu-item index="align-center" @click="emit('menu-action', 'edit.align', { type: 'center' })">
            居中对齐
          </el-menu-item>
          <el-menu-item index="align-right" @click="emit('menu-action', 'edit.align', { type: 'right' })">
            右对齐
          </el-menu-item>
        </el-sub-menu>
        <el-divider />
        <el-menu-item index="edit-enable" @click="emit('menu-action', 'edit.enable')">
          启用
        </el-menu-item>
        <el-menu-item index="edit-disable" @click="emit('menu-action', 'edit.disable')">
          禁用
        </el-menu-item>
        <el-menu-item index="edit-bypass" @click="emit('menu-action', 'edit.bypass')">
          旁路
        </el-menu-item>
        <el-divider />
        <el-menu-item index="edit-preferences" @click="emit('menu-action', 'edit.preferences')">
          <el-icon><Setting /></el-icon>
          偏好设置
        </el-menu-item>
      </el-sub-menu>

      <!-- 视图菜单 -->
      <el-sub-menu index="view">
        <template #title>视图</template>
        <el-menu-item index="view-block-library" @click="emit('menu-action', 'view.toggleBlockLibrary')">
          显示左侧模块树面板
        </el-menu-item>
        <el-menu-item index="view-console" @click="emit('menu-action', 'view.toggleConsole')">
          显示控制台面板
        </el-menu-item>
        <el-menu-item index="view-console-lock" @click="emit('menu-action', 'view.consoleLock')">
          控制台滚动锁定
        </el-menu-item>
        <el-menu-item index="view-save-console" @click="emit('menu-action', 'view.saveConsole')">
          保存控制台
        </el-menu-item>
        <el-menu-item index="view-clear-console" @click="emit('menu-action', 'view.clearConsole')">
          清空控制台
        </el-menu-item>
        <el-divider />
        <el-menu-item index="view-variables" @click="emit('menu-action', 'view.toggleVariables')">
          显示变量面板
        </el-menu-item>
        <el-menu-item index="view-expressions" @click="emit('menu-action', 'view.showExpressions')">
          显示模块参数表达式
        </el-menu-item>
        <el-menu-item index="view-values" @click="emit('menu-action', 'view.showValues')">
          显示模块参数值
        </el-menu-item>
        <el-menu-item index="view-hide-disabled" @click="emit('menu-action', 'view.hideDisabled')">
          隐藏禁用模块
        </el-menu-item>
        <el-menu-item index="view-auto-hide-ports" @click="emit('menu-action', 'view.autoHidePorts')">
          自动隐藏端口标签
        </el-menu-item>
        <el-menu-item index="view-show-comments" @click="emit('menu-action', 'view.showComments')">
          显示模块注释
        </el-menu-item>
        <el-menu-item index="view-show-ids" @click="emit('menu-action', 'view.showIds')">
          显示所有模块 ID
        </el-menu-item>
        <el-divider />
        <el-menu-item index="view-code-preview" @click="emit('menu-action', 'view.codePreview')">
          生成代码预览
        </el-menu-item>
        <el-divider />
        <el-menu-item index="view-zoom-in" @click="emit('menu-action', 'view.zoomIn')">
          <el-icon><ZoomIn /></el-icon>
          放大
        </el-menu-item>
        <el-menu-item index="view-zoom-out" @click="emit('menu-action', 'view.zoomOut')">
          <el-icon><ZoomOut /></el-icon>
          缩小
        </el-menu-item>
        <el-menu-item index="view-reset-zoom" @click="emit('menu-action', 'view.resetZoom')">
          重置缩放
        </el-menu-item>
        <el-divider />
        <el-menu-item index="view-flow-errors" @click="emit('menu-action', 'view.flowErrors')">
          流图错误
        </el-menu-item>
        <el-menu-item index="view-find-blocks" @click="emit('menu-action', 'view.findBlocks')">
          <el-icon><Search /></el-icon>
          查找模块
        </el-menu-item>
      </el-sub-menu>

      <!-- 运行菜单 -->
      <el-sub-menu index="run">
        <template #title>运行</template>
        <el-menu-item index="run-generate" @click="emit('menu-action', 'run.generate')">
          <el-icon><Cpu /></el-icon>
          生成
        </el-menu-item>
        <el-menu-item index="run-execute" @click="emit('menu-action', 'run.execute')">
          <el-icon><VideoPlay /></el-icon>
          执行
        </el-menu-item>
        <el-menu-item index="run-terminate" @click="emit('menu-action', 'run.terminate')">
          <el-icon><VideoPause /></el-icon>
          终止
        </el-menu-item>
      </el-sub-menu>

      <!-- 帮助菜单 -->
      <el-sub-menu index="help">
        <template #title>帮助</template>
        <el-menu-item index="help-about" @click="emit('menu-action', 'help.about')">
          <el-icon><InfoFilled /></el-icon>
          关于
        </el-menu-item>
      </el-sub-menu>
    </el-menu>
  </div>
</template>

<script setup lang="ts">
import {
  FolderOpened, Document, Camera, RefreshLeft, RefreshRight,
  CopyDocument, DocumentCopy, Delete, Setting,
  ZoomIn, ZoomOut, Search, Cpu, VideoPlay, VideoPause, InfoFilled
} from '@element-plus/icons-vue'

// 定义事件
const emit = defineEmits<{
  'menu-action': [action: string, data?: any]
}>()
</script>

<style scoped>
.menu-bar {
  border-bottom: 1px solid #e4e7ed;
}

.main-menu {
  border-bottom: none;
}

.main-menu :deep(.el-menu-item),
.main-menu :deep(.el-sub-menu__title) {
  height: 40px;
  line-height: 40px;
  padding: 0 15px;
}

.main-menu :deep(.el-sub-menu .el-menu-item) {
  height: 36px;
  line-height: 36px;
  padding: 0 20px;
}

.main-menu :deep(.el-divider--horizontal) {
  margin: 8px 0;
}
</style>