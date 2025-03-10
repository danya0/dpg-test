<script lang="ts">
	/**
	 * Dependencies
	 */
	import PostsApi from '$lib/api/methods/posts';
	import { getReasonPhrase } from 'http-status-codes';

	/**
	 * Components
	 */
	import Feed from '$lib/components/feed/Feed.svelte';
	import Loader from '$lib/components/shared/Loader.svelte';
	import ErrorComponent from '$lib/components/shared/Error.svelte';
	import Notification from '$lib/components/shared/Notification.svelte';
	import type { IPost } from '$lib/api/types/models/post';

	/**
	 * Constants
	 */
	const LOCAL_STORAGE_KEY = 'cachedPosts';

	/**
	 * State
	 */
	let posts: IPost[] = [];
	let isLoading: boolean = true;
	let error: string | null = null;
	let showCacheNotification: boolean = false;

	/**
	 * Load data
	 * @description Метод для получения постов
	 */
	const loadData = async (): Promise<void> => {
		try {
			// Пытаемся загрузить данные из сети
			const response = await PostsApi.getAll();

			if (!response.success) {
				throw new Error(getReasonPhrase(response.data.code));
			}

			// Сохраняем данные в локальное хранилище
			const fetchedPosts = response.data;
			localStorage.setItem(LOCAL_STORAGE_KEY, JSON.stringify(fetchedPosts));
			posts = fetchedPosts;
		} catch (e) {
			//  При возникновении ошибки пытаемся загрузить данные из кеша
			const cachedPosts: string | null = localStorage.getItem(LOCAL_STORAGE_KEY);

			if (cachedPosts) {
				posts = JSON.parse(cachedPosts) as IPost[];
				showCacheNotification = true;
			} else {
				error = 'Не удалось получить данные';
			}
		} finally {
			isLoading = false;
		}
	};

	loadData();
</script>

{#if isLoading}
	<Loader />
{:else if error}
	<ErrorComponent>
		{error}
	</ErrorComponent>
{:else}
	{#if showCacheNotification}
		<Notification>Данные загружены из кеша</Notification>
	{/if}
	<Feed {posts} />
{/if}
