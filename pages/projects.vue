<template>
	<main>
		<section class="header projects">
			<h1>Overview</h1>
			<div v-if="!projects.length" class="no-project">
				<p class="projects-overview__empty">
					It seems like you are not part of any proect yet. Create one or ask permission in an existing project!
				</p>
				<nuxt-link to="/project/create" class="button projects__new-project">
					Create new project
				</nuxt-link>
			</div>
			<div v-if="projects.length" class="projects__navigation">
				<ul class="projects__sort">
					<li v-for="(item, index) of sortOptions" :key="index" class="projects__sort-item">
						<button :class="{ active: sortMethod === item }" class="projects__sort-button" @click="changeSorting(item)">
							{{ item }}
						</button>
					</li>
				</ul>
				<nuxt-link to="/project/create" class="button projects__new-project">
					Create new project
				</nuxt-link>
			</div>
		</section>
		<section class="projects-overview">
			<ul v-if="projects.length" class="projects-overview__list row">
				<li
					v-for="(item, index) of projects"
					:key="index"
					class="projects-overview__item column small-full medium-half large-third"
				>
					<div class="single-project">
						<nuxt-link :to="`/project/${item.id}`" class="single-project__url">
							{{ item.name }}
						</nuxt-link>
						<span class="single-project__process-bar">
							<span class="single-project__process-bar-status" :style="`width: ${calculateProcess(item)}%`"></span>
						</span>
						<h3 class="single-project__title">
							{{ item.name }}
						</h3>
						<p class="grey">
							<span class="format">
								Updated:
							</span>
							<span class="value">
								{{ formatDate(item.updatedAt) }}
							</span>
						</p>
						<p class="grey">
							<span class="format">
								Created:
							</span>
							<span class="value">
								{{ formatDate(item.createdAt) }}
							</span>
						</p>
						<ul class="single-project__users">
							<li v-for="(user, i) of item.users" :key="i" class="single-project__user">
								<img :src="user.image" :alt="`Profile picture of ${user.name}`" />
								<span class="single-project__name">
									{{ user.firstName }}
								</span>
							</li>
						</ul>
					</div>
				</li>
			</ul>
		</section>
	</main>
</template>

<script>
export default {
	middleware: 'session',
	data() {
		return {
			sortMethod: 'Created',
			sortOptions: ['Created', 'Last updated', 'Name'],
			projects: []
		};
	},
	async asyncData({ app }) {
		const response = await app.$axios({
			method: 'get',
			withCredentials: true,
			url: '/projects'
		});
		if (response && response.data) {
			return {
				projects: response.data
			};
		}
	},
	created() {
		this.changeSorting('Created');
	},
	methods: {
		changeSorting(val) {
			this.sortMethod = val;
			const projects = this.projects;
			if (val === 'Name') {
				projects.sort((a, b) => {
					const nameA = a.name.toUpperCase();
					const nameB = b.name.toUpperCase();
					if (nameA < nameB) {
						return -1;
					}
					if (nameA > nameB) {
						return 1;
					}

					// names must be equal
					return 0;
				});
			} else if (val === 'Created') {
				projects.sort((a, b) => {
					const val1 = new Date(a.createdAt);
					const val2 = new Date(b.createdAt); // ignore upper and lowercase
					if (val1.getTime() < val2.getTime()) {
						return 1;
					}
					if (val1.getTime() > val2.getTime()) {
						return -1;
					}
					return 0;
				});
			} else if (val === 'Last updated') {
				projects.sort((a, b) => {
					const val1 = new Date(a.updatedAt);
					const val2 = new Date(b.updatedAt); // ignore upper and lowercase
					if (val1.getTime() < val2.getTime()) {
						return 1;
					}
					if (val1.getTime() > val2.getTime()) {
						return -1;
					}
					return 0;
				});
			}
			this.projects = projects;
		},
		calculateProcess(project) {
			const check = ['colorStatus', 'gridStatus', 'typoStatus', 'fontStatus'];

			let complete = 0;
			check.forEach((name) => {
				if (project[name]) {
					complete++;
				}
			});
			return ((complete / check.length) * 100) | 0;
		},
		formatDate(oldDate) {
			const date = new Date(`${oldDate}`);
			const now = new Date();
			const diff = Math.abs(date - now) / 1000 / 60; //diff is in minutes
			if (diff < 60) {
				if (Math.floor(diff) <= 1) {
					return `1 minute ago`;
				} else {
					return `${Math.floor(diff)} minutes ago`;
				}
			} else if (diff < 1440) {
				const hours = Math.floor(diff / 60);
				if (hours <= 1) {
					return `1 hour ago`;
				} else {
					return `${hours} hours ago`;
				}
			} else {
				const days = Math.floor(diff / 1440);
				if (days <= 1) {
					return `1 day ago`;
				} else {
					return `${days} days ago`;
				}
			}
		}
	}
};
</script>
<style lang="scss">
@import '~tools';
.no-project {
	display: flex;
	flex-direction: column;
	justify-content: center;
	align-items: center;
}
.projects {
	display: flex;
	flex-direction: column;
	align-items: flex-start;
	padding: grid(0 2);

	&__navigation {
		display: flex;
		justify-content: space-between;
		align-items: center;
		width: 100%;
		margin-top: rem(60);
	}
	&__new-project {
		white-space: nowrap;
	}
	&__sort {
		width: 100%;
		margin: 0;
		&-button {
			box-shadow: none;
			border: none;
			outline: none;
			background: transparent;
			color: color(ParlorBlack);
			font-weight: 500;
			font-size: rem(16);
			opacity: 0.5;
			transition: 0.15s ease-in-out;
			cursor: pointer;
			padding: rem(0 40 0 0);
			&.active {
				text-decoration: underline;
				opacity: 1;
			}
			&:hover {
				opacity: 1;
			}
		}
	}
	&-overview {
		padding: grid(0 1.5);
		&__empty {
			margin: grid(1 0.5 0);
			font-weight: 500;
		}
		&__item {
			padding: grid(0.5);
		}
		&__list {
			margin-top: grid(0.5);
		}
	}
}

.single-project {
	position: relative;
	display: flex;
	flex-direction: column;
	width: grid(6);
	height: grid(3);
	background: color(Gallery, 0.5);
	transition: 0.3s $easing;
	padding: rem(20 30);
	&:hover {
		box-shadow: rem(0 0 10 0) color(ParlorBlack, 0.1);
	}
	.grey {
		display: inline-block;
		font-size: rem(14);
		line-height: 1;
		& + .grey {
			margin-top: rem(10);
		}
	}
	.value {
		font-weight: 600;
	}
	&__title {
		position: relative;
		z-index: 1;
		margin-top: rem(15);
	}
	&__url {
		position: absolute;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		text-indent: -9999rem;
		overflow: hidden;
	}
	&__users {
		position: absolute;
		bottom: rem(20);
		display: flex;
	}
	&__name {
		height: 0;
		color: color(ParlorBlack);
		font-size: rem(12);
		white-space: nowrap;
		opacity: 0;
		transition: opacity 0.2s $easing;
	}
	&__user {
		position: relative;
		width: rem(25);
		height: rem(25);
		border-radius: 50%;
		margin-right: rem(10);
		&:hover {
			.single-project__name {
				opacity: 1;
			}
		}
		img {
			width: 100%;
			height: 100%;
			border-radius: 50%;
			object-fit: cover;
		}
	}
	&__process-bar {
		position: relative;
		display: inline-block;
		width: 100%;
		height: rem(4);
		border-radius: 5px;
		background: color(Grey, 0.2);
		&-status {
			position: relative;
			z-index: 1;
			display: block;
			width: 0;
			height: 100%;
			border-radius: 5px;
			background: color(Parlor);
		}
	}
}
</style>
