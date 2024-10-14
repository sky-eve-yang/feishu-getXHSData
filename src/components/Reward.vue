<template>
    <div class="reward">
        <el-popover :visible="visible" placement="top" :offset="12" :width="232" :height="268.6" trigger="click">
            <div class="reward-popover">
                <div class="reward-popover-author">
                    <span class="left">作者</span>
                    <span class="name">{{ author }}</span>
                </div>
                <div class="reward-popover-tips">扫码请Ta喝杯咖啡</div>
                <div class="reward-popover-img">
                    <img v-if="qrCode" :src="qrCode" alt="qrCode">
                    <i v-else class="el-icon-loading"></i>
                </div>
            </div>
            <template #reference>
                <el-button style="padding: 6px 12px;border: 1px solid #0064fa;border-radius: 20px;color: #0064fa" slot="reference"><span style="display: flex;flex-direction: row;align-items: center;justify-content: center;" role="img" aria-label="heart_stroked" class="semi-icon semi-icon-default semi-icon-heart_stroked"><svg viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg" width="1em" height="1em" focusable="false" aria-hidden="true"><path fill-rule="evenodd" clip-rule="evenodd" d="M12 4.15622C11.0091 2.88867 9.61824 2.22282 8.1766 2.08357C6.31414 1.90368 4.36427 2.59816 3.04873 3.94814C1.6489 5.38462 0.968851 7.13942 1.00109 9.02554C1.03296 10.8897 1.75737 12.8138 3.03631 14.6511C5.05479 17.551 9.25556 21.0323 10.7729 22.2494C11.4924 22.8265 12.5076 22.8265 13.2271 22.2494C14.7444 21.0323 18.9452 17.551 20.9637 14.6511C22.2426 12.8138 22.967 10.8897 22.9989 9.02555C23.0312 7.13942 22.3511 5.38463 20.9513 3.94814C19.6358 2.59816 17.6859 1.90368 15.8234 2.08357C14.3818 2.22282 12.9908 2.88866 12 4.15622ZM7.98432 4.07431C6.7137 3.95158 5.36504 4.43688 4.4811 5.34396C3.44916 6.40293 2.97778 7.64465 3.0008 8.99136C3.0242 10.36 3.56175 11.9052 4.6778 13.5085C6.49479 16.1189 10.4282 19.4076 12 20.6697C13.5718 19.4076 17.5052 16.1189 19.3222 13.5085C20.4382 11.9052 20.9758 10.36 20.9992 8.99135C21.0222 7.64464 20.5509 6.40292 19.5189 5.34396C18.635 4.43688 17.2863 3.95158 16.0157 4.07431C14.7891 4.19278 13.6393 4.87262 13.0227 6.3452C12.6441 7.24958 11.3559 7.2496 10.9772 6.3452C10.3607 4.87261 9.21086 4.19278 7.98432 4.07431Z" fill="currentColor"></path></svg></span><span class="reward-btn-tip">赞赏作者</span></el-button>
            </template>
        </el-popover>
    </div>
</template>

<script>
export default {
    name: 'Reward',
    props: {
        author: {
            type: String,
            required: true
        },
        qrCode: {
            type: String,
            required: true
        }
    }
};
</script>

<style scoped>
.reward {
    width: 100%;
    padding: 80px 0 10px 0;
    display: flex;
    justify-content: center;
    align-items: center;
}

.el-button--primary.el-button--outline {
    border: 1px solid #0064fa;
    border-radius: 20px;
}

.el-button:focus, .el-button:hover {
    background-color: rgba(46, 50, 56, 0.05) !important;
}

.el-popover {
    padding: 12px 16px;
}

.reward-popover-author {
    height: 22px;
    line-height: 22px;
    font-size: 14px;
}

.reward-popover-author .left {
    color: #6b7075;
}

.reward-popover-author .name {
    color: #1c1f23;
    margin-left: 16px;
}

.reward-popover-tips {
    color: #888d92;
    font-size: 12px;
    text-align: center;
    height: 20px;
    line-height: 20px;
    margin-bottom: 4px;
}

.reward-popover-img {
    width: 200px;
    height: 200px;
    background-color: #e6e8ea;
}

.reward-popover-img img {
    display: block;
    width: 100%;
    height: 100%;
}

.reward-btn-tip {
    margin-left: 8px;
    font-size: 14px;
    color: #0064fa;
    font-weight: bold;
}

</style>