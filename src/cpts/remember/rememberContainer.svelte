<script lang="ts">
  import LevelOpt from './LevelOpt.svelte';
  import {
    hideTi,
    initArr,
    recordTiResult,
    rememberData,
    setRoundStatus,
    switchGame,
    startShowTi,
    triggerNextAction,
    switchModal,
  } from "../../stores/rememberStore";
  import Counter from "../public/counter.svelte";
  import Timer from "../public/timer.svelte";
  import Blocks from "./blocks.svelte";
    import ErrorIndicator from "./ErrorIndicator.svelte";
    import { sleep } from '../../funcs/common';
    import Modal from '../public/modal/modalContainer.svelte';
    import ConfirmModal from '../public/modal/confirm.svelte';
    import InfoModal from '../public/modal/info.svelte'
    $:opt='简单'
    
    const showTxt = { idle: "Start", running: "Stop", pending: "Resume",success:"done" };
  const startAction = async () => {
    if($rememberData.status==='idle'){
      switchGame();
      initArr();
      startShowTi();
      await sleep(1000);
      hideTi();
      setRoundStatus("running");
    }else{
      $rememberData.status='idle'
      initArr()
    }
  };
  const onConfirmed=async()=>{
    switchModal()
    await sleep(1000)
    setRoundStatus("running");
    triggerNextAction();
    startShowTi();
    await sleep(1000);
    hideTi();
  }
  const onCancel=async()=>{
    switchModal()
    $rememberData.status='idle'
      initArr()
  }
  const timeoutFuc = async () => {
    setRoundStatus("fail");
    recordTiResult(false);
    startShowTi();
    await sleep(1000);
    hideTi();
    await sleep(1000);
    if($rememberData.current!==$rememberData.total){
      switchModal()
    }
  };
  const successFuc = async () => {
    
    setRoundStatus("idle");
    await sleep(1000)
    recordTiResult(true);
    hideTi()
    await sleep(1000);
    if($rememberData.current!==$rememberData.total){
      switchModal()
    }
    
  };
</script>

<div class="text-xs w-full sm:text-lg">
  <div class="flex justify-between items-center my-1 px-2">
    <LevelOpt ></LevelOpt>
    <button class="btnPrimary h-8" on:click={startAction}>{showTxt[$rememberData.status]}</button>
  </div>
  <div class="flex justify-between items-center my-1 px-2">
   <ErrorIndicator />
    <Counter total={$rememberData.total} idx={$rememberData.current} />
    
    <Timer
      duration={$rememberData.roundTime}
      operation={$rememberData.roundStatus}
      on:success={successFuc}
      on:timeout={timeoutFuc}
    />
    
  </div>
  <Blocks
    tis={$rememberData.arr[$rememberData.current - 1]}
  />

</div>
<Modal isShow={$rememberData.isShowModal} on:onClosed={()=>switchModal()}>
  <ConfirmModal title={"Confirm"} content={"Go ahead to next?"} on:onPositive={onConfirmed} on:onNegative={onCancel}/>
</Modal>
