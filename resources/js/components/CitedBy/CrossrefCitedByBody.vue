<template>
	<div :class="cn('root')">
		<p :class="cn('count')">
			{{
				t('plugins.generic.crossref.citedBy.citationCount', {
					count: store.total,
				})
			}}
		</p>

		<div :class="cn('citationsWrapper')">
			<ul :class="cn('citationsList')">
				<li
					v-for="(citation, index) in store.citations"
					:key="index"
					:class="cn('citationsListItem')"
				>
					<div :class="cn('citationsListItemContent')">
						<h3 :class="cn('citationTitle')">
							{{ citation.title }}
						</h3>

						<p :class="cn('citationAuthors')">
							{{ citation.authors }}
						</p>

						<p :class="cn('citationSource')">
							<span>
								<template
									v-for="(source, index) in getSourceLine(citation)"
									:key="index"
								>
									<span>{{ source }}</span>
									<span
										:class="cn('citationSourceDelimiter')"
										v-if="index < getSourceLine(citation).length - 1"
									>
										.
									</span>
								</template>
							</span>
						</p>

						<p :class="cn('citationDoi')">
							<a
								v-if="citation.doi"
								:href="store.getDoiExternalLink(citation.doi)"
								target="_blank"
								rel="noopener noreferrer"
							>
								{{ `doi.org/${citation.doi}` }}
							</a>
						</p>
					</div>
				</li>
			</ul>
		</div>

		<div :class="cn('actions')">
			<PkpButton
				:class="cn('root')"
				:is-disabled="store.isLoading || store.total < 1"
				@click="store.copyAllToClipboard()"
			>
				{{
					store.copiedToClipboard
						? t('common.copied')
						: t('plugins.generic.crossref.citedBy.copyCitationDetails')
				}}
			</PkpButton>

			<PkpButton :class="cn('root')" @click="onClose" :is-secondary="true">
				{{ t('common.close') }}
			</PkpButton>
		</div>
	</div>
</template>

<script setup>
import {useCrossrefCitedByStore} from './useCrossrefCitedByStore.js';

const {usePkpLocalize} = pkp.modules.usePkpLocalize;
const {t} = usePkpLocalize();
const {usePkpStyles} = pkp.modules.usePkpStyles;

const props = defineProps({
	styles: {type: Object, default: () => ({})},
	onClose: {type: Function, default: () => () => {}},
});

const {cn} = usePkpStyles('CrossrefCitedByBody', props.styles);
const store = useCrossrefCitedByStore();

function getSourceLine(citation) {
	const source = [
		citation?.journal,
		citation?.institutionName,
		citation.year,
		getSourceLocator(citation),
	];

	return source.filter(Boolean);
}

function getSourceLocator(citation) {
	const parts = [];

	if (citation.volume) {
		parts.push(
			citation.issue
				? t('plugins.generic.crossref.citedBy.citationSource.volumeWithIssue', {
						volume: citation.volume,
						issue: citation.issue,
					})
				: t('plugins.generic.crossref.citedBy.citationSource.volume', {
						volume: citation.volume,
					}),
		);
	} else if (citation.issue) {
		parts.push(
			t('plugins.generic.crossref.citedBy.citationSource.issueWithoutVolume', {
				issue: citation.issue,
			}),
		);
	}

	if (citation.firstPage) {
		parts.push(
			t('plugins.generic.crossref.citedBy.citationSource.firstPage', {
				page: citation.firstPage,
			}),
		);
	}

	return parts.join(', ');
}
</script>
