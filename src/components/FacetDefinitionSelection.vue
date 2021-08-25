<!-- Copyright (c) 2021 MintJams Inc. Licensed under MIT License. -->

<template>
	<div class="h-100 position-relative overflow-hidden">
		<div class="h-100 overflow-hidden" v-on:click.prevent.stop>
			<div class="position-relative w-100">
				<input type="text" class="pl-2 pr-4 py-2 border-none outline-none material material-blue w-100 text-shadow" name="filterText" v-model="filterText" v-focus.delay="200" autocomplete="off">
				<div class="absolute-0 left-auto pr-2 text-small text-shadow c-pointer justify-content-center align-items-center" :class="{'d-flex': filterText, 'd-none': !filterText}" v-on:click.prevent.stop="clearFilterText">
					<i class="fas fa-times"></i>
				</div>
			</div>
			<div class="mt-1 overflow-hidden position-relative" style="height: calc(100% - 2.75rem);">
				<perfect-scrollbar>
					<div v-if="filteredFacetDefinitions.length == 0" class="p-2 text-small text-upper font-weight-semibold text-black-50 text-shadow text-truncate">No matching records found.</div>
					<div v-for="facetDefinition in filteredFacetDefinitions" :key="facetDefinition.key" class="list-item d-flex justify-content-start align-items-center overflow-hidden c-pointer bg-hover" v-on:click.prevent.stop="doSelect(facetDefinition)">
						<div class="icon-block text-black-50 text-shadow"><i :class="facetIconClasses(facetDefinition)"><i class="fas fa-plus overlay-top-right"></i></i></div>
						<div class="flex-grow-1 pr-2 font-weight-semibold text-shadow text-truncate">
							<span v-if="facetDefinition.label">{{facetDefinition.label}}<span class="ml-2 text-black-50 text-small">{{facetDefinition.key}}</span></span>
							<span v-if="!facetDefinition.label">{{facetDefinition.key}}</span>
						</div>
					</div>
				</perfect-scrollbar>
			</div>
		</div>
	</div>
</template>

<script>
/* eslint-disable no-console */
import { PerfectScrollbar } from "vue2-perfect-scrollbar";
import commons from '@mintjamsinc/webtop-app-commons';
const {Facets, Strings} = commons;

export default {
	props: {
		'excludes': {
			'type': Array,
			'default': []
		},
	},
	components: {
		PerfectScrollbar: PerfectScrollbar
	},
	data() {
		return {
			'filterText': '',
			'value': null,
			'facetDefinitionIndex': [],
			'facetDefinitions': {},
		};
	},
	created() {
		let vm = this;
		class UI {
			constructor() {
				this.$onChanged = function() {
					return;
				};
			}

			get value() {
				return vm.value;
			}
			set value(value) {
				vm.value = value;
			}

			get facetDefinitions() {
				return vm.facetDefinitions;
			}

			get onChanged() {
				return this.$onChanged;
			}
			set onChanged(f) {
				this.$onChanged = f;
			}
		}
		vm.ui = new UI();

		window.Webtop.cmsClient.listFacetDefinitions().then(function(response) {
			for (let facetDefinition of response.facetDefinitions) {
				vm.facetDefinitionIndex.push(facetDefinition.key);
				vm.facetDefinitions[facetDefinition.key] = facetDefinition.$data;
			}
		});
	},
	computed: {
		filteredFacetDefinitions() {
			let vm = this;
			let filters = {'filterText': vm.filterText};

			let results = [];
			for (let key of vm.facetDefinitionIndex) {
				let fd = vm.facetDefinitions[key];

				// excludes
				if (vm.excludes.indexOf(key) != -1) {
					continue;
				}
				if (fd.type == 'propertyset') {
					let excludesAll = true;
					for (let k of fd.keys) {
						if (vm.facetDefinitionIndex.indexOf(k) != -1 && vm.excludes.indexOf(k) == -1) {
							excludesAll = false;
							break;
						}
					}
					if (excludesAll) {
						continue;
					}
				}

				// name
				if (filters.filterText) {
					let key = fd.key.toLowerCase();
					let label = Strings.defaultString(fd.label).toLowerCase();
					let words = filters.filterText.trim().split(" ");
					let match = true;
					for (let word of words) {
						if (key.indexOf(word.toLowerCase()) == -1 && label.indexOf(word.toLowerCase()) == -1) {
							match = false;
							break;
						}
					}
					if (!match) {
						continue;
					}
				}

				results.push(fd);
			}

			const sortKey = function(fd) {
				let key = fd.key;
				let label = Strings.defaultString(fd.label);
				if (label) {
					return label;
				}
				return key;
			};

			results.sort(function(a, b) {
				let ak = sortKey(a);
				let bk = sortKey(b);
				return Facets.compareString(ak, bk);
			});

			return results;
		}
	},
	methods: {
		clearFilterText() {
			let vm = this;
			vm.filterText = '';
			vm.focusFilterText();
		},
		focusFilterText() {
			let vm = this;
			vm.$nextTick(function() {
				vm.$el.querySelector("input[name='filterText']").focus();
			});
		},
		facetIconClasses(facetDefinition) {
			return Facets.iconClasses(facetDefinition.type);
		},
		doSelect(fd) {
			let vm = this;
			if (!fd) {
				return;
			}
			vm.value = fd;
			vm.ui.$onChanged();
		},
	}
}
</script>
