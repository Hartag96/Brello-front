<script>
	import { onMount, afterUpdate, beforeUpdate } from 'svelte';
  import { cardStore } from '../stores/cardStore.js';
  import { userStore } from '../stores/userStore.js';
  import userIcon from "../../public/images/userIcon.png"
  import menuIcon from "../../public/images/menuIcon.png"
  import checklistIcon from "../../public/images/checklist.png"
  import checkedIcon from "../../public/images/tick.png"
  import labelsIcon from "../../public/images/labelsIcon.png"
  import duedateIcon from "../../public/images/duedateIcon.png"
  import moveIcon from "../../public/images/moveIcon.png"
  import copyIcon from "../../public/images/copyIcon.png"
  import watchIcon from "../../public/images/watchIcon.png"
  import archiveIcon from "../../public/images/archiveIcon.png"
  import shareIcon from "../../public/images/shareIcon.png"
  import attachmentIcon from "../../public/images/attachmentIcon.png"
	import Loader from "./common/Loader.svelte";
	import Tasks from "./Tasks.svelte";
	import moment from 'moment';
	import constants from '../constants/constants';
	let promise;
	let newCommentBody = "";
	let newCardDescriptiony = "";
	let newTaskListName = "";
	let addTaskListMode = false;

  let store
	let currentUser;
  onMount(()=> {
    const unsubscribeCard = cardStore.subscribe(async (value) => {
      store = value
    })
    const unsubscribeUser = userStore.subscribe(async (value) => {
      currentUser = value
    })
  })

  const handleCloaseBox = (e) => {
    if(e.target.id === "bgClose" || e.target.id === "btnClose"){
      promise = cardStore.hideModal()
			return true;
    }
  }
	// TODO: impement
	const handleUpdateDescription = (e) => {
		console.log(e.target);
		// const file = e.target.files[0]
	}

	const handleFileAttach = (e) => {
		const file = e.target.files[0]
		console.log(file);
		const fileName = file.name.substr(0, file.name.lastIndexOf("."))
		const fileFormat = file.name.substr(file.name.lastIndexOf(".")+1)
		promise = cardStore.uploadAttachment(file, fileName, fileFormat, store.card.id).then(() => {
			promise = cardStore.openCard(store.card.id)
		})
	}

	const handleRemoveAttachment = (attachmentId) => {
		promise = cardStore.removeAttachment(attachmentId).then(() => {
			promise = cardStore.openCard(store.card.id)
		})
	}

	const handleAddCommentAttachment = (e, commentId) => {
		const file = e.target.files[0]
		console.log(file);
		const fileName = file.name.substr(0, file.name.lastIndexOf("."))
		const fileFormat = file.name.substr(file.name.lastIndexOf(".")+1)
		promise = cardStore.uploadAttachment(file, fileName, fileFormat, store.card.id).then(() => {
			newCommentBody = `[${fileName}]`
		})
	}

	const handleSaveComment = (e) => {
		if(newCommentBody !== ""){
			promise = cardStore.saveComment(store.card.id, currentUser.userId, newCommentBody).then(() => {
				newCommentBody = ""
				promise = cardStore.openCard(store.card.id)
			})
		}
	}

	const handleCreateTaskList = () => {
		promise = cardStore.createTaskList(newTaskListName, store.card.id).then(() => {
			newTaskListName = ""
			addTaskListMode = false
			promise = cardStore.openCard(store.card.id)
		})
	}
</script>

{#if store && store.visible}
	{#await promise}
		<Loader />
	{:then nothing}
		{#if store.isLoading }
			<Loader />
		{:else}
	  <div
	    id="bgClose"
	    on:click={handleCloaseBox}
	    class="flex flex-row justify-center fixed top-0 pt-20 w-full z-20"
	  >
	    <div class="hide-scrollbar w-3/4 bg-indigo-500 py-8 px-4 pb-10 text-white  overflow-y-auto border-8 border-indigo-500 ">
	      <button
	        class="w-100 float-right font-bold -mt-6"
	        id="btnClose">
	          x
	        </button>
	      <div class="flex flex-row items-start justify-start items-start pb-20">

	        <div class="w-4/5 mr-2 ">
	          <!-- HEADER -->
	          <div class="w-full flex flex-row">
	            <div class="w-1/12 pt-2">
	              <img src={userIcon}/>
	            </div>
	            <div class="w-11/12 ml-2">
	              <p class="font-bold text-2xl"> {store.card.cardName} </p>
	              <p class="font-bold">
	                in list <a class="underline">{store.card.list_name}</a>
	              </p>
	              <div class="flex flex-row mt-8 text-xs">
	                <div class="mr-8">
	                  <p>MEMBERS</p>
	                  <div class="my-2 font-bold text-xl text-gray-800">
	                    <button class="mr-2 bg-indigo-300 px-3 py-1 rounded-full"> D </button>
	                    <button class="mr-2 bg-indigo-200 px-3 py-1 rounded-full"> + </button>
	                  </div>
	                </div>
	                <div class="">
	                  <p>LABELS</p>
	                  <div class="my-2 font-bold text-xl">
	                    <button class="mr-1 bg-green-500 px-3 py-1  rounded-sm"> FRONT-END </button>
	                    <button class="mr-1 bg-indigo-200 text-gray-800 px-3 py-1 rounded-sm"> + </button>
	                  </div>
	                </div>
	              </div>
	            </div>
	          </div>

	          <!-- DESCRIPTION -->
	          <div class="w-full flex flex-row mt-10">
	            <div class="w-1/12 pt-1">
	              <img src={menuIcon}/>
	            </div>
	            <div class="w-11/12 ml-2">
	              <p class="font-bold text-2xl inline-block"> Description </p>
	              <button class="mr-1 bg-indigo-200 text-gray-800 ml-2 -mb-1 px-3 py-1 rounded-sm"> EDIT </button>
	              <div class="w-full mt-4 mr-4">
	                {store.card.descriptionCard}
	              </div>
	            </div>
	          </div>

	          <!-- ATTACHMENT -->
						{#if (store.card.attachments.length !== 0)}
		          <div class="w-full flex flex-row mt-10">
		            <div class="w-1/12 pt-1">
		              <img src={attachmentIcon}/>
		            </div>
		            <div class="w-11/12 ml-2">
		              <p class="font-bold text-2xl inline-block"> Attachments </p>
									{#each store.card.attachments as attachment}
			              <div class=" w-full mt-4 mr-4 ">
											<div class="flex flex-row flex-no-wrap w-4/5 mb-2">
												<a
													class="bg-indigo-400 h-full p-5 font-bold text-center"
													href={`${attachment.url}${constants.ATTACHMENT_TOKEN}`}
													target="_blank"
												>
													{attachment.formatUrl.toUpperCase()}
												</a>
												<div class="w-5/6 p-2 truncate bg-indigo-300 font-bold">
													<p>{attachment.nameUrl}</p>
													<button on:click={() => handleRemoveAttachment(attachment.id)} class="text-xs underline">DELETE</button>
												</div>
											</div>
			              </div>
									{/each}
		            </div>
		          </div>
						{/if}

	          <!-- CHECKLIST -->
						{#if store.card.tasklists}
							<Tasks/>
						{/if}

	          <!-- ACTIVITY -->
	          <div class="w-full flex flex-row mt-10">
	            <div class="w-1/12 pt-1">
	              <img src={menuIcon}/>
	            </div>
	            <div class="w-11/12 ml-2">
	              <p class="font-bold text-2xl inline-block"> Activity </p>
	              <div class=" w-full mt-4 mr-4">
									<div class="flex flex-row -ml-16 mr-8 my-8">
										<button class="mr-2 bg-indigo-300 px-3 py-1 rounded-full h-full" title={currentUser.username}> {currentUser.username.charAt(0).toUpperCase()} </button>
										<div class="ml-6 w-4/5">
											<input
												bind:value={newCommentBody}
												class="px-4 py-2 w-full text-gray-800 bg-indigo-100"
												placeholder="Write a commnet"
											/>
											<div class="w-full bg-indigo-100">
												<button
													class="text-xs py-2 px-4 m-2 {newCommentBody.length === 0 ? "bg-indigo-200" : "bg-indigo-700"}"
													on:click={handleSaveComment}
													disabled={newCommentBody.length === 0}
												>
													Save
												</button>
												<label
													class="bg-indigo-700 float-right py-2 px-4 m-2 cursor-pointer"
													for="commentAttachment"
												>
													<img src={attachmentIcon} class="h-4 float-right ">
												</label>
												<input class="hidden" type="file" name="commentAttachment" id="commentAttachment" on:change={handleAddCommentAttachment}/>
											</div>
										</div>
									</div>
									{#if store.card.comments}
										{#each store.card.comments as comment}
											<div class="my-4">
												<div class="flex flex-row">
												<!-- {comment.idUser.charAt(0).toUpperCase()} -->
													<button class="-ml-16 mr-8 bg-indigo-300 px-3 py-1 rounded-full" title={comment.username}> {comment.username.charAt(0).toUpperCase()} </button>
													<span class="font-bold">{comment.username}</span>
													<span class="ml-3">{moment(comment.published_date).fromNow()}</span>
												</div>
												<p class="bg-indigo-400 p-2 my-2 w-4/5">
												{comment.body}
												</p>
											</div>
										{/each}
									{/if}
	              </div>
	            </div>
	          </div>

	        </div>


	        <div class="w-1/5 pl-2">
	          <span class="block text-sm ">ADD TO CARD</span>
	          <button class="w-full bg-indigo-600 block p-2 my-2">
	            <img src={userIcon} class="inline-block w-5 float-left mt-1">
	            <span class="float-left ml-2">Join</span>
	          </button>
	          <button class="w-full bg-indigo-600 block p-2 my-2">
	            <img src={userIcon} class="inline-block w-5 float-left mt-1">
	            <span class="float-left ml-2">Members</span>
	          </button>
	          <button class="w-full bg-indigo-600 block p-2 my-2">
	            <img src={labelsIcon} class="inline-block w-5 float-left mt-1">
	            <span class="float-left ml-2">Labels</span>
	          </button>
	          <button
								class="w-full bg-indigo-600 block p-2 my-2"
								on:click={() => addTaskListMode = true}
							>
	            <img src={checklistIcon} class="inline-block w-5 float-left mt-1">
	            <span class="float-left ml-2">Checklist</span>
	          </button>
						{#if addTaskListMode}
							<div>
								<div
									on:click|stopPropagation={() => addTaskListMode = false}
									class="{`absolute w-full h-screen top-0 left-0 z-40 ${(addTaskListMode) ? 'block' : 'hidden'}`}">
									.
								</div>

								<div class="relative absolute z-50 w-64 ml-1">
									<div
										class="pt-1"
									>
										New Checklist name:
									</div>
									<input
										class="bg-indigo-300 px-2 py-1 my-2 w-2/3"
										bind:value={newTaskListName}
									/>
									<button
										class="px-4 py-2 bg-indigo-700  my-1 text-center"
										on:click={handleCreateTaskList}
									>
										Add checklist
									</button>
								</div>
							</div>
						{/if}
	          <button class="w-full bg-indigo-600 block p-2 my-2">
	            <img src={duedateIcon} class="inline-block w-5 float-left mt-1">
	            <span class="float-left ml-2">Due Date</span>
	          </button>
	          <button class="w-full bg-indigo-600 block p-2 my-2">
	            <img src={attachmentIcon} class="inline-block w-5 float-left mt-1">
							<input class="hidden" type="file" name="attachment" id="attachment" on:change={handleFileAttach}/>
							<label class="float-left ml-2" for="attachment"> Attachment </label>
	          </button>

	          <span class="block text-sm mt-6">ACTIONS</span>
	          <button class="w-full bg-indigo-600 block p-2 my-2">
	            <img src={moveIcon} class="inline-block w-5 float-left mt-1">
	            <span class="float-left ml-2">Move</span>
	          </button>
	          <button class="w-full bg-indigo-600 block p-2 my-2">
	            <img src={copyIcon} class="inline-block w-5 float-left mt-1">
	            <span class="float-left ml-2">Copy</span>
	          </button>
	          <button class="w-full bg-indigo-600 block p-2 my-2">
	            <img src={watchIcon} class="inline-block w-5 float-left mt-1">
	            <span class="float-left ml-2">Watch</span>
	          </button>
	          <button class="w-full bg-indigo-600 block p-2 my-2">
	            <img src={archiveIcon} class="inline-block w-5 float-left mt-1">
	            <span class="float-left ml-2">Archive</span>
	          </button>
	          <button class="w-full bg-indigo-600 block p-2 my-2">
	            <img src={shareIcon} class="inline-block w-5 float-left mt-1">
	            <span class="float-left ml-2">Share</span>
	          </button>
	        </div>
	      </div>

	    </div>
	  </div>
		{/if}
	  <div
	    id="bgClose"
	    on:click={handleCloaseBox}
	    class="bg-gray-300 opacity-50 z-10 w-full h-full fixed top-0"
	  >
	  </div>
	{:catch error}
		<p style="color: red">{error.message}</p>
	{/await}
{/if}

<style>

button:focus, input:focus {
	outline: 0
}

.hide-scrollbar {
	height: 80vh;
	overflow: -moz-scrollbars-none;
	scrollbar-width: none;
}
.hide-scrollbar::-webkit-scrollbar {
  display: none;
}
textarea::placeholder {
	color: #edf2f7;
}
</style>
