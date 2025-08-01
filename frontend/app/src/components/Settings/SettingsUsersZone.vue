<template>
	<div class="col">
		<div class="bg-body-tertiary rounded p-3 shadow-sm">
			<div class="row row-gap-3">
				<div class="col-lg-4 col-md-12">
					<!-- add user button -->
					<a class="w-100 btn btn-primary" href="#" role="button" data-bs-toggle="modal"
						data-bs-target="#addUserModal">{{ $t("settingsUsersZone.buttonAddUser") }}</a>

					<!-- Modal add user -->
					<UsersAddEditUserModalComponent :action="'add'" @createdUser="addUserList"
						@isLoadingNewUser="setIsLoadingNewUser" />
				</div>
				<!-- form to search-->
				<div class="col">
					<form class="d-flex">
						<input class="form-control me-2" type="text" name="userUsername"
							:placeholder='$t("settingsUsersZone.labelSearchUsersByUsername")' v-model="searchUsername"
							required>
					</form>
				</div>
			</div>
			<div>
				<LoadingComponent v-if="isLoading" />
				<div v-else>
					<!-- Checking if usersArray is loaded and has length -->
					<div class="mt-3" v-if="usersArray && usersArray.length">
						<!-- title zone -->
						<span>{{ $t("settingsUsersZone.labelNumberOfUsers1") }}{{ usersNumber }}{{
							$t("settingsUsersZone.labelNumberOfUsers2") }}{{ usersArray.length }}{{
								$t("settingsUsersZone.labelNumberOfUsers3") }}</span>

						<!-- Displaying loading new gear if applicable -->
						<ul class="list-group list-group-flush" v-if="isLoadingNewUser">
							<li class="list-group-item rounded">
								<LoadingComponent />
							</li>
						</ul>

						<!-- Displaying loading if gears are updating -->
						<LoadingComponent v-if="isUsersUpdatingLoading" />

						<!-- list zone -->
						<ul class="list-group list-group-flush" v-for="user in usersArray" :key="user.id" :user="user"
							v-else>
							<UsersListComponent :user="user" @userDeleted="updateUserList" @editedUser="editUserList" />
						</ul>

						<!-- pagination area -->
						<PaginationComponent :totalPages="totalPages" :pageNumber="pageNumber"
							@pageNumberChanged="setPageNumber" v-if="!searchUsername" />
					</div>
					<!-- Displaying a message or component when there are no activities -->
					<div v-else>
						<br>
						<NoItemsFoundComponent />
					</div>
				</div>
			</div>
		</div>
	</div>
</template>

<script setup>
import { ref, onMounted, watch } from "vue";
import { useI18n } from "vue-i18n";
import { push } from "notivue";
import { debounce } from "lodash";
import LoadingComponent from "@/components/GeneralComponents/LoadingComponent.vue";
import NoItemsFoundComponent from "@/components/GeneralComponents/NoItemsFoundComponents.vue";
import UsersListComponent from "@/components/Settings/SettingsUsersZone/UsersListComponent.vue";
import PaginationComponent from "@/components/GeneralComponents/PaginationComponent.vue";
import UsersAddEditUserModalComponent from "@/components/Settings/SettingsUsersZone/UsersAddEditUserModalComponent.vue";
import { users } from "@/services/usersService";

const { t } = useI18n();
const isLoading = ref(true);
const isUsersUpdatingLoading = ref(false);
const isLoadingNewUser = ref(false);
const usersArray = ref([]);
const usersNumber = ref(0);
const pageNumber = ref(1);
const numRecords = 5;
const totalPages = ref(1);
const searchUsername = ref("");

const performSearch = debounce(async () => {
	if (!searchUsername.value) {
		pageNumber.value = 1;
		await fetchUsers();
		return;
	}
	try {
		usersArray.value = await users.getUserContainsUsername(searchUsername.value);
	} catch (error) {
		push.error(`${t("settingsUsersZone.errorFetchingUsers")} - ${error}`);
	}
}, 500);

function setPageNumber(page) {
	pageNumber.value = page;
}

async function updateUsers() {
	try {
		isUsersUpdatingLoading.value = true;
		usersArray.value = await users.getUsersWithPagination(
			pageNumber.value,
			numRecords,
		);
		isUsersUpdatingLoading.value = false;
	} catch (error) {
		push.error(`${t("settingsUsersZone.errorFetchingUsers")} - ${error}`);
	}
}

async function fetchUsers() {
	try {
		updateUsers();
		usersNumber.value = await users.getUsersNumber();
		totalPages.value = Math.ceil(usersNumber.value / numRecords);
	} catch (error) {
		push.error(`${t("settingsUsersZone.errorFetchingUsers")} - ${error}`);
	}
}

function updateUserList(userDeletedId) {
	usersArray.value = usersArray.value.filter(
		(user) => user.id !== userDeletedId,
	);
	usersNumber.value--;
	push.success(t("settingsUsersZone.successUserDeleted"));
}

function addUserList(createdUser) {
	usersArray.value.unshift(createdUser);
	usersNumber.value++;
}

function editUserList(editedUser) {
	const index = usersArray.value.findIndex((user) => user.id === editedUser.id);
	usersArray.value[index] = editedUser;
}

function setIsLoadingNewUser(state) {
	isLoadingNewUser.value = state;
}

onMounted(async () => {
	await fetchUsers();
	isLoading.value = false;
});

watch(searchUsername, performSearch, { immediate: false });
watch(pageNumber, updateUsers, { immediate: false });
</script>