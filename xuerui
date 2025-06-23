#!/bin/bash

# 脚本名称：Cysic 验证程序一键管理脚本
# 作者：Andy甘 (推特@mingfei2022)
# 版本：1.1

# 设置脚本在遇到错误时退出
set -e

# 定义常量路径
VERIFIER_DIR="$HOME/cysic-verifier"
CYSIC_KEYS_DIR="$HOME/.cysic/keys"
AUTO_RECONNECT_PID_FILE="/tmp/cysic_auto_reconnect.pid"
AUTO_RECONNECT_LOG_FILE="$HOME/cysic_auto_reconnect.log"
VERIFIER_DOWNLOAD_URL="https://github.com/cysic-labs/cysic-phase3/releases/download/v1.0.0/verifier_linux"
SETUP_SCRIPT_URL="https://github.com/cysic-labs/cysic-phase3/releases/download/v1.0.0/setup_linux.sh"

# 颜色定义
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[0;33m'
NC='\033[0m' # No Color

# 辅助函数：启动验证程序在一个新的 screen 会话中
start_verifier_in_screen() {
    if [ ! -d "$VERIFIER_DIR" ]; then
        echo -e "${RED}错误：无法找到验证程序目录 ($VERIFIER_DIR)。请确保已安装节点。${NC}"
        return 1
    fi

    cd "$VERIFIER_DIR" || { echo -e "${RED}错误：无法切换到 $VERIFIER_DIR 目录。${NC}"; return 1; }

    # 终止现有的 screen 会话（如果存在）
    if screen -list | grep -q "cysic_verifier"; then
        echo -e "${YELLOW}检测到现有 screen 会话 'cysic_verifier'，正在终止...${NC}"
        screen -S cysic_verifier -X quit || true # `|| true` prevents script from exiting if screen fails to quit gracefully
        sleep 1 # 稍微等待 screen 进程结束
    fi

    echo -e "${GREEN}正在新的 screen 会话 'cysic_verifier' 中启动验证程序...${NC}"
    screen -dmS cysic_verifier bash start.sh
    if [ $? -ne 0 ]; then
        echo -e "${RED}错误：启动验证程序失败。请检查 $VERIFIER_DIR/start.sh 脚本。${NC}"
        echo -e "${YELLOW}提示：如果看到'err: rpc error'，请等待几分钟，验证程序将尝试连接。${NC}"
        return 1
    else
        echo -e "${GREEN}验证程序已在 screen 会话 'cysic_verifier' 中成功启动！${NC}"
        echo -e "您可以通过选项 2 或运行 '${YELLOW}screen -r cysic_verifier${NC}' 进入会话查看实时状态。"
        echo -e "按 ${YELLOW}Ctrl+A${NC} 然后按 ${YELLOW}D${NC} 退出 screen 会话而不终止程序。"
        return 0
    fi
}

# 辅助函数：禁用自动重新连接
disable_auto_reconnect() {
    echo -e "${YELLOW}正在检查并禁用自动重新连接...${NC}"
    if [ -f "$AUTO_RECONNECT_PID_FILE" ]; then
        PID=$(cat "$AUTO_RECONNECT_PID_FILE")
        if ps -p "$PID" > /dev/null 2>&1; then # Check if PID is still active
            kill "$PID" 2>/dev/null || true # Kill the process gracefully
            echo -e "${GREEN}已终止 PID $PID 对应的自动重新连接进程。${NC}"
        else
            echo -e "${YELLOW}自动重新连接 PID 文件 ($PID) 存在，但进程已不存在。${NC}"
        fi
        rm -f "$AUTO_RECONNECT_PID_FILE" || true
    else
        echo -e "${YELLOW}未找到自动重新连接的 PID 文件，无需禁用。${NC}"
    fi
}

# 函数：安装并运行节点
install_and_run_node() {
    echo -e "${YELLOW}请输入您的奖励地址（以 0x 开头）：${NC}"
    read -r REWARD_ADDRESS

    if [ -z "$REWARD_ADDRESS" ]; then
        echo -e "${RED}错误：奖励地址不能为空。${NC}"
        return 1
    fi

    echo -e "${GREEN}正在使用奖励地址 $REWARD_ADDRESS 下载并运行安装脚本...${NC}"
    curl -L "$SETUP_SCRIPT_URL" -o "$HOME/setup_linux.sh"
    if [ $? -ne 0 ]; then
        echo -e "${RED}错误：无法下载安装脚本。请检查网络连接或URL。${NC}"
        return 1
    fi

    chmod +x "$HOME/setup_linux.sh"
    "$HOME/setup_linux.sh" "$REWARD_ADDRESS"
    if [ $? -ne 0 ]; then
        echo -e "${RED}错误：安装脚本执行失败。请检查错误信息。${NC}"
        return 1
    fi

    echo -e "${YELLOW}等待安装完成，尝试启动验证程序...${NC}"
    sleep 5 # 给 setup_linux.sh 一点时间完成文件写入

    start_verifier_in_screen
    if [ $? -eq 0 ]; then
        echo -e "${GREEN}重要：您的助记词文件已生成，位于 ${YELLOW}$CYSIC_KEYS_DIR/${NC} 文件夹。"
        echo -e "请务必${RED}妥善备份此文件夹中的文件${NC}，否则您将无法再次运行验证程序。"
    fi
}

# 函数：查看 screen 日志
view_logs() {
    if screen -list | grep -q "cysic_verifier"; then
        echo -e "${GREEN}找到 screen 会话 'cysic_verifier'。${NC}"
        echo -e "您可以直接进入 screen 会话查看实时日志，命令为：${YELLOW}screen -r cysic_verifier${NC}"
        echo -e "按 ${YELLOW}Ctrl+A${NC} 然后按 ${YELLOW}D${NC} 退出 screen 会话而不终止程序。"
        echo -e "${YELLOW}是否要进入 screen 会话查看日志？（y/n）${NC}"
        read -r enter_screen
        if [ "$enter_screen" = "y" ] || [ "$enter_screen" = "Y" ]; then
            screen -r cysic_verifier
        else
            echo -e "${YELLOW}您选择了不进入 screen 会话。可以通过 'screen -r cysic_verifier' 随时查看。${NC}"
        fi
    else
        echo -e "${RED}错误：未找到名为 'cysic_verifier' 的 screen 会话。${NC}"
        echo -e "${YELLOW}请确保已通过选项 1 启动验证程序，或检查 screen 会话是否已被终止。${NC}"
        echo -e "您可以运行 '${YELLOW}screen -list${NC}' 查看所有活动 screen 会话。"
    fi
}

# 函数：重新连接验证程序子菜单
reconnect_verifier() {
    while true; do
        clear
        echo -e "=== ${GREEN}重新连接验证程序子菜单${NC} ==="
        echo "1. 手动重新连接验证程序"
        echo "2. 启用每小时自动重新连接"
        echo "3. 禁用自动重新连接"
        echo "4. 返回主菜单"
        echo -e "${YELLOW}请输入您的选择（1-4）：${NC}"
        read -r sub_choice

        case $sub_choice in
            1)
                echo -e "${YELLOW}正在尝试手动重新连接验证程序...${NC}"
                disable_auto_reconnect # 确保手动连接时不会有旧的自动连接在干扰
                start_verifier_in_screen
                echo -e "${YELLOW}按 Enter 键返回子菜单...${NC}"
                read -r
                ;;
            2)
                echo -e "${YELLOW}启用每小时自动重新连接验证程序...${NC}"
                disable_auto_reconnect # 确保只运行一个自动连接实例

                echo -e "自动重新连接将记录在 ${YELLOW}$AUTO_RECONNECT_LOG_FILE${NC}"
                # 启动后台循环
                (
                    # Log file header
                    echo "--- Auto Reconnect Log Started on $(date) ---" >> "$AUTO_RECONNECT_LOG_FILE"
                    while true; do
                        echo -e "[$(date)] ${YELLOW}尝试自动重新连接...${NC}" | tee -a "$AUTO_RECONNECT_LOG_FILE"
                        if start_verifier_in_screen; then
                            echo -e "[$(date)] ${GREEN}自动重新连接成功。${NC}" | tee -a "$AUTO_RECONNECT_LOG_FILE"
                        else
                            echo -e "[$(date)] ${RED}自动重新连接失败。${NC}" | tee -a "$AUTO_RECONNECT_LOG_FILE"
                        fi
                        echo -e "[$(date)] ${YELLOW}等待 1 小时后再次检查...${NC}" | tee -a "$AUTO_RECONNECT_LOG_FILE"
                        sleep 3600 # 每小时（3600秒）执行一次
                    done
                ) &
                PID=$!
                echo "$PID" > "$AUTO_RECONNECT_PID_FILE"
                echo -e "${GREEN}自动重新连接已在后台启用 (PID: $PID)。日志可在 ${YELLOW}$AUTO_RECONNECT_LOG_FILE${NC} 查看。${NC}"
                echo -e "${YELLOW}按 Enter 键返回子菜单...${NC}"
                read -r
                ;;
            3)
                disable_auto_reconnect
                echo -e "${YELLOW}按 Enter 键返回子菜单...${NC}"
                read -r
                ;;
            4)
                return 0
                ;;
            *)
                echo -e "${RED}无效的选择，请输入 1-4。${NC}"
                echo -e "${YELLOW}按 Enter 键继续...${NC}"
                read -r
                ;;
        esac
    done
}

# 函数：更新Cysic验证程序
update_verifier() {
    echo -e "=== ${GREEN}更新 Cysic 验证程序${NC} ==="
    # 1. 检查节点是否已安装
    if [ ! -d "$VERIFIER_DIR" ]; then
        echo -e "${RED}错误：Cysic 验证程序尚未安装。请先选择选项 1 安装节点。${NC}"
        return 1
    fi

    echo -e "${YELLOW}警告：更新操作将${RED}终止${YELLOW}当前运行的验证程序，${RED}下载最新版本${YELLOW}并${RED}修改配置文件${YELLOW}。${NC}"
    echo -e "如果已启用自动重新连接，它将被临时禁用并随新版本启动后重新激活。"
    echo -e "${YELLOW}是否继续？（y/n）${NC}"
    read -r confirm
    if [ "$confirm" != "y" ] && [ "$confirm" != "Y" ]; then
        echo -e "${YELLOW}更新操作已取消。${NC}"
        return 0
    fi

    # 禁用自动重新连接，防止更新时其启动旧版本
    disable_auto_reconnect

    # 停止 screen 会话 (start_verifier_in_screen 内部会处理终止，这里是为了更明确的步骤说明)
    if screen -list | grep -q "cysic_verifier"; then
        echo -e "${YELLOW}正在终止 'cysic_verifier' screen 会话...${NC}"
        screen -S cysic_verifier -X quit || true
        sleep 1
    fi

    echo -e "${YELLOW}进入验证程序目录...${NC}"
    cd "$VERIFIER_DIR" || { echo -e "${RED}错误：无法切换到 $VERIFIER_DIR 目录。${NC}"; return 1; }

    echo -e "${YELLOW}正在更新 config.yaml 中的 verify_endpoint...${NC}"
    sed -i 's#^  verify_endpoint:.*#  verify_endpoint: "http://verifier-rpc.prover.xyz:50052"#' config.yaml
    if [ $? -ne 0 ]; then
        echo -e "${YELLOW}警告：config.yaml 修改失败或未发生更改，请手动检查。${NC}"
    else
        echo -e "${GREEN}config.yaml 已更新。${NC}"
    fi

    echo -e "${YELLOW}正在删除旧的 verifier binary 和 data 目录...${NC}"
    rm -f verifier # -f ensures it works even if not present, and doesn't ask
    rm -rf data
    if [ $? -ne 0 ]; then
        echo -e "${YELLOW}警告：无法删除旧的 verifier binary 或 data 目录，尝试继续。${NC}"
    fi

    echo -e "${YELLOW}正在下载新的 verifier binary...${NC}"
    curl -L "$VERIFIER_DOWNLOAD_URL" -o verifier
    if [ $? -ne 0 ]; then
        echo -e "${RED}错误：无法下载新的 verifier binary。请检查网络连接或URL。${NC}"
        return 1
    fi

    echo -e "${YELLOW}正在设置新的 verifier binary 为可执行文件...${NC}"
    chmod +x verifier
    if [ $? -ne 0 ]; then
        echo -e "${RED}错误：无法设置 verifier 为可执行文件。请检查权限。${NC}"
        return 1
    fi

    echo -e "${GREEN}验证程序文件已成功更新。现在重新启动...${NC}"
    start_verifier_in_screen

    echo -e "${GREEN}Cysic 验证程序更新并重新启动完成。${NC}"
    echo -e "请使用选项 2 查看日志确认运行状态。"
}

# 函数：删除会话和节点（保留 ~/.cysic/keys/ 文件夹）
delete_session_and_node() {
    echo -e "${RED}警告：此操作将终止 'cysic_verifier' screen 会话并删除 $VERIFIER_DIR 目录。${NC}"
    echo -e "${GREEN}注意：$CYSIC_KEYS_DIR 文件夹（包含助记词）将被${YELLOW}保留${NC}。${NC}"
    echo -e "如果启用了自动重新连接，也将被禁用。"
    echo -e "${YELLOW}是否继续？（y/n）${NC}"
    read -r confirm
    if [ "$confirm" = "y" ] || [ "$confirm" = "Y" ]; then
        disable_auto_reconnect

        # 检查并终止 screen 会话
        if screen -list | grep -q "cysic_verifier"; then
            screen -S cysic_verifier -X quit || true
            echo -e "${GREEN}已终止 'cysic_verifier' screen 会话。${NC}"
        else
            echo -e "${YELLOW}未找到 'cysic_verifier' screen 会话，跳过终止步骤。${NC}"
        fi

        # 删除 ~/cysic-verifier/ 目录
        if [ -d "$VERIFIER_DIR" ]; then
            rm -rf "$VERIFIER_DIR"
            if [ $? -eq 0 ]; then
                echo -e "${GREEN}已成功删除 $VERIFIER_DIR 目录。${NC}"
            else
                echo -e "${RED}错误：删除 $VERIFIER_DIR 目录失败，请检查权限或目录状态。${NC}"
                return 1
            fi
        else
            echo -e "${YELLOW}未找到 $VERIFIER_DIR 目录，跳过删除步骤。${NC}"
        fi

        # 确认 ~/.cysic/keys/ 文件夹保留
        if [ -d "$CYSIC_KEYS_DIR" ]; then
            echo -e "${GREEN}已保留 $CYSIC_KEYS_DIR 文件夹，请确保已备份其中的助记词文件。${NC}"
        else
            echo -e "${YELLOW}警告：未找到 $CYSIC_KEYS_DIR 文件夹，可能尚未生成助记词。${NC}"
        fi
        echo -e "${GREEN}删除操作完成。${NC}"
    else
        echo -e "${YELLOW}操作已取消，未删除任何内容。${NC}"
    fi
}

# 主菜单循环
while true; do
    clear
    echo -e "============================================"
    echo -e "${GREEN}=== 脚本由Andy甘免费开源，推特@mingfei2022 ===${NC}"
    echo -e "${YELLOW}=== Cysic 验证程序管理菜单 ===${NC}"
    echo -e "============================================"
    echo -e "${GREEN}1. 安装并运行节点${NC}"
    echo -e "${GREEN}2. 查看 screen 日志${NC}"
    echo -e "${GREEN}3. 重新连接验证程序（手动/自动每小时）${NC}"
    echo -e "${GREEN}4. 更新 Cysic 验证程序${NC}" # 新增选项
    echo -e "${GREEN}5. 删除会话和节点（保留 keys 文件夹）${NC}"
    echo -e "${RED}6. 退出脚本${NC}"
    echo -e "============================================"
    echo -e "${YELLOW}请输入您的选择（1-6）：${NC}"
    read -r choice

    case $choice in
        1)
            install_and_run_node
            echo -e "${YELLOW}按 Enter 键返回菜单...${NC}"
            read -r
            ;;
        2)
            view_logs
            echo -e "${YELLOW}按 Enter 键返回菜单...${NC}"
            read -r
            ;;
        3)
            reconnect_verifier
            ;;
        4) # 新增的选项处理
            update_verifier
            echo -e "${YELLOW}按 Enter 键返回菜单...${NC}"
            read -r
            ;;
        5)
            delete_session_and_node
            echo -e "${YELLOW}按 Enter 键返回菜单...${NC}"
            read -r
            ;;
        6)
            echo -e "${YELLOW}退出脚本...${NC}"
            exit 0
            ;;
        *)
            echo -e "${RED}无效的选择，请输入 1-6。${NC}"
            echo -e "${YELLOW}按 Enter 键继续...${NC}"
            read -r
            ;;
    esac
done
