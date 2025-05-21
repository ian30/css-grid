<script lang="ts">
	import { onMount } from 'svelte';
	import Tooltip from '$lib/Tooltip.svelte';
	let isResizing = false;
	let startX: number;
	let leftPanelWidth = 300; // Set default width
	let container: HTMLElement;
	let mounted = false;
	let codePreview = '';
	let codeElement: HTMLElement;

	// Modify the gridItems type and initialization
	interface GridItem {
		id: number;
		content: string;
		area?: string;
		background?: string;
		columnSpan: number;
		rowSpan: number;
	}

	// Add preset layouts
	const presetLayouts = {
		simple: {
			areas: `
				'header header content-1'
				'sidebar Content-3 content-1'
				'footer footer footer'
			`,
			items: [
				{ id: 1, content: 'header', area: 'header', background: '#8B5CF6' },
				{ id: 2, content: 'sidebar', area: 'sidebar', background: '#2DD4BF' },
				{
					id: 3,
					content: 'Content-1',
					area: 'content-1',
					background: 'linear-gradient(45deg, #8B5CF6, #2DD4BF)'
				},
				{
					id: 4,
					content: 'Content-3',
					area: 'Content-3',
					background: 'linear-gradient(45deg, #8B5CF6, #2DD4BF)'
				},
				{ id: 5, content: 'footer', area: 'footer', background: '#8B5CF6' }
			]
		},
		dashboard: {
			areas: `
				'nav nav nav nav'
				'side main main widgets'
				'side main main widgets'
				'side stats stats widgets'
				'side media chat widgets'
			`,
			items: [
				{ id: 1, content: 'Navigation', area: 'nav', background: '#3B82F6' },
				{ id: 2, content: 'Sidebar', area: 'side', background: '#1E40AF' },
				{
					id: 3,
					content: 'Main Content',
					area: 'main',
					background: 'linear-gradient(135deg, #3B82F6, #2DD4BF)'
				},
				{ id: 4, content: 'Statistics', area: 'stats', background: '#0EA5E9' },
				{ id: 5, content: 'Media', area: 'media', background: '#0D9488' },
				{ id: 6, content: 'Chat', area: 'chat', background: '#0891B2' },
				{
					id: 7,
					content: 'Widgets',
					area: 'widgets',
					background: 'linear-gradient(135deg, #1E40AF, #0D9488)'
				}
			]
		},
		magazine: {
			areas: `
				'featured featured featured sidebar1'
				'article1 article2 article3 sidebar1'
				'main main main sidebar2'
				'gallery gallery shop sidebar2'
				'footer footer footer footer'
			`,
			items: [
				{ id: 1, content: 'Featured Story', area: 'featured', background: '#EC4899' },
				{ id: 2, content: 'Article 1', area: 'article1', background: '#DB2777' },
				{ id: 3, content: 'Article 2', area: 'article2', background: '#BE185D' },
				{ id: 4, content: 'Article 3', area: 'article3', background: '#9D174D' },
				{
					id: 5,
					content: 'Main Story',
					area: 'main',
					background: 'linear-gradient(135deg, #EC4899, #8B5CF6)'
				},
				{ id: 6, content: 'Sidebar Ads', area: 'sidebar1', background: '#A21CAF' },
				{ id: 7, content: 'Recent Posts', area: 'sidebar2', background: '#7E22CE' },
				{ id: 8, content: 'Gallery', area: 'gallery', background: '#6B21A8' },
				{ id: 9, content: 'Shop', area: 'shop', background: '#581C87' },
				{ id: 10, content: 'Footer', area: 'footer', background: '#4C1D95' }
			]
		},
		megaComplex: {
			areas: `
				'h1 h1 h2 h2 h3 h3 h4 h4'
				'h5 h6 h7 h8 h9 h10 h11 h12'
				'm1 m1 m2 m2 m3 m3 m4 m4'
				'm5 m6 m7 m8 m9 m10 m11 m12'
				'm13 m13 m14 m14 m15 m15 m16 m16'
				'm17 m18 m19 m20 m21 m22 m23 m24'
				'm25 m25 m26 m26 m27 m27 m28 m28'
				'm29 m30 m31 m32 m33 m34 m35 m36'
				'f1 f1 f2 f2 f3 f3 f4 f4'
			`,
			items: Array.from({ length: 40 }, (_, i) => {
				const row = i < 12 ? 'h' : i < 36 ? 'm' : 'f';
				const num = i < 12 ? i + 1 : i < 36 ? i - 11 : i - 35;
				const area = `${row}${num}`;

				// Create gradient colors based on position
				const hue1 = (i * 20) % 360;
				const hue2 = (hue1 + 40) % 360;
				const background = `linear-gradient(${i * 15}deg, hsl(${hue1}, 70%, 60%), hsl(${hue2}, 70%, 60%))`;

				return {
					id: i + 1,
					content: `Item ${i + 1}`,
					area,
					background
				};
			})
		}
	};

	// Start with an empty grid on load
	let currentLayout: keyof typeof presetLayouts = 'simple';
	let gridItems: GridItem[] = [];
	let nextId = 1;

	// Grid Container Properties
	let gridProperties = {
		// Template Properties
		gridTemplateColumns: '1fr 2fr 1fr',
		gridTemplateRows: 'auto',
		gridTemplateAreas: '',

		// Gap Properties
		columnGap: '1rem',
		rowGap: '1rem',

		// Auto Properties
		gridAutoColumns: 'auto',
		gridAutoRows: 'auto',
		gridAutoFlow: 'row',

		// Alignment Properties (Container)
		justifyContent: 'start',
		alignContent: 'start',
		placeContent: 'start',
		justifyItems: 'stretch',
		alignItems: 'stretch',
		placeItems: 'stretch'
	};
	let pendingGridTemplateAreas = '';

	// Property Options
	const propertyOptions = {
		gridAutoFlow: ['row', 'column', 'row dense', 'column dense'],
		justifyContent: [
			'start',
			'end',
			'center',
			'stretch',
			'space-around',
			'space-between',
			'space-evenly'
		],
		alignContent: [
			'start',
			'end',
			'center',
			'stretch',
			'space-around',
			'space-between',
			'space-evenly'
		],
		justifyItems: ['start', 'end', 'center', 'stretch'],
		alignItems: ['start', 'end', 'center', 'stretch']
	};

	// Column Templates
	let columnCount = 3;
	let columnUnit = 'fr';
	let columnSize = 1;

	// Add to the top of the script section with other state variables
	let columnGapSize = 1;
	let columnGapUnit = 'rem';
	let rowGapSize = 1;
	let rowGapUnit = 'rem';

	// Grid property explanations
	const propertyExplanations = {
		columns:
			"Define the number and size of columns in your grid. Use 'fr' for fractional units, 'px' for pixels, or '%' for percentage of container width.",
		templateAreas:
			'Create named grid areas for easy placement of items. Use quotes for each row and name each cell to create a visual layout map.',
		columnGap: 'Set the space between grid columns. Use any valid CSS unit (px, rem, em, etc).',
		rowGap: 'Set the space between grid rows. Use any valid CSS unit (px, rem, em, etc).',
		autoFlow:
			"Control how items are automatically placed in the grid. 'row'/'column' for direction, add 'dense' to fill gaps.",
		autoColumns:
			'Define the size of automatically generated columns. Applies when items are placed outside defined grid areas.',
		autoRows:
			'Define the size of automatically generated rows. Applies when items are placed outside defined grid areas.',
		justifyContent:
			'Control alignment of the entire grid within its container along the horizontal axis.',
		alignContent:
			'Control alignment of the entire grid within its container along the vertical axis.',
		justifyItems:
			'Set default horizontal alignment for all grid items. Can be overridden by individual items.',
		alignItems:
			'Set default vertical alignment for all grid items. Can be overridden by individual items.'
	};

	// Remove drag and drop state variables and functions
	let draggedItem: GridItem | null = null;
	let draggedOverItem: GridItem | null = null;
	let dropPreviewElement: HTMLElement | null = null;

	function updateColumns() {
		if (columnCount > 0) {
			gridProperties.gridTemplateColumns = `repeat(${columnCount}, ${columnSize}${columnUnit})`;
			updatePreviewStyles();
		}
	}

	function startResize(event: MouseEvent) {
		if (!mounted) return;
		isResizing = true;
		startX = event.pageX;
		leftPanelWidth = container.querySelector('.left-panel')?.getBoundingClientRect().width ?? 300;

		window.addEventListener('mousemove', handleMouseMove);
		window.addEventListener('mouseup', stopResize);
	}

	function handleMouseMove(event: MouseEvent) {
		if (!isResizing || !mounted) return;

		const diff = event.pageX - startX;
		const newWidth = Math.max(200, Math.min(leftPanelWidth + diff, window.innerWidth - 200));
		container.style.gridTemplateColumns = `${newWidth}px 8px 1fr`;
	}

	function stopResize() {
		isResizing = false;
		window.removeEventListener('mousemove', handleMouseMove);
		window.removeEventListener('mouseup', stopResize);
	}

	function addGridItem() {
		const newItem: GridItem = {
			id: nextId,
			content: `Content ${nextId}`,
			background: 'linear-gradient(45deg, #8B5CF6, #2DD4BF)',
			columnSpan: 1,
			rowSpan: 1
		};
		gridItems = [...gridItems, newItem];
		nextId++;
	}

	function updatePreviewStyles() {
		console.log('Updating preview styles');
		const previewContainer = document.querySelector('.preview-container') as HTMLElement;
		if (previewContainer) {
			// Ensure grid display is set
			previewContainer.style.display = 'grid';

			Object.entries(gridProperties).forEach(([property, value]) => {
				if (value !== '') {
					// Convert property names to modern CSS Grid property names
					const cssProperty =
						property === 'columnGap' ? 'column-gap' : property === 'rowGap' ? 'row-gap' : property;
					console.log(`Setting ${cssProperty} to ${value}`);
					(previewContainer.style as any)[cssProperty] = value;
				}
			});

			// Ensure grid template columns is set
			if (!previewContainer.style.gridTemplateColumns) {
				previewContainer.style.gridTemplateColumns = gridProperties.gridTemplateColumns;
			}

			codePreview = generateCSSCode();
			// Highlight the code after updating
			if (typeof Prism !== 'undefined' && codeElement) {
				codeElement.innerHTML = Prism.highlight(codePreview, Prism.languages.css, 'css');
			}
		}
	}

	function generateCSSCode(): string {
		const propertyMap: { [key: string]: string } = {
			gridTemplateColumns: 'grid-template-columns',
			gridTemplateRows: 'grid-template-rows',
			gridTemplateAreas: 'grid-template-areas',
			columnGap: 'column-gap',
			rowGap: 'row-gap',
			gridAutoColumns: 'grid-auto-columns',
			gridAutoRows: 'grid-auto-rows',
			gridAutoFlow: 'grid-auto-flow',
			justifyContent: 'justify-content',
			alignContent: 'align-content',
			placeContent: 'place-content',
			justifyItems: 'justify-items',
			alignItems: 'align-items',
			placeItems: 'place-items'
		};

		const css = Object.entries(gridProperties)
			.filter(([_, value]) => value !== '') // Filter out empty values
			.map(([property, value]) => {
				const cssProperty = propertyMap[property] || property;
				return `  ${cssProperty}: ${value};`;
			})
			.join('\n');

		return `.container {\n  display: grid;\n${css}\n}`;
	}

	function copyToClipboard() {
		navigator.clipboard.writeText(codePreview);
	}

	// Add this function to handle gap updates
	function updateGaps() {
		gridProperties.columnGap = `${columnGapSize}${columnGapUnit}`;
		gridProperties.rowGap = `${rowGapSize}${rowGapUnit}`;
		updatePreviewStyles();
	}

	// Add function to apply preset layout
	function applyPresetLayout(preset: keyof typeof presetLayouts) {
		const layout = presetLayouts[preset];
		gridItems = layout.items.map((item) => ({
			...item,
			columnSpan: 1,
			rowSpan: 1
		}));
		nextId = gridItems.length + 1;
		gridProperties.gridTemplateAreas = layout.areas.trim();
		gridProperties.gridTemplateColumns = '1fr 2fr 1fr';
		updatePreviewStyles();
	}

	// Add delete function
	function deleteGridItem(idToDelete: number) {
		gridItems = gridItems.filter((item) => item.id !== idToDelete);
	}

	function clearAll() {
		gridItems = [];
		gridProperties.gridTemplateAreas = '';
		updatePreviewStyles();
	}

	$: {
		if (mounted) {
			updatePreviewStyles();
		}
	}

	// Add reactive statements to watch gap changes
	$: {
		if (mounted && (gridProperties.columnGap || gridProperties.rowGap)) {
			console.log('Gap values changed:', gridProperties.columnGap, gridProperties.rowGap);
			updatePreviewStyles();
		}
	}

	onMount(() => {
		if (container) {
			container.style.gridTemplateColumns = `${leftPanelWidth}px 8px 1fr`;
			mounted = true;
			// Initialize grid properties immediately
			requestAnimationFrame(() => {
				updatePreviewStyles();
			});
		}
	});

	// Add resize-related state and functions
	function handleResize(event: MouseEvent, item: GridItem, direction: 'right' | 'bottom') {
		event.stopPropagation();
		const startX = event.pageX;
		const startY = event.pageY;
		const initialColumnSpan = item.columnSpan;
		const initialRowSpan = item.rowSpan;
		const gridItem = document.querySelector(`[data-id="${item.id}"]`) as HTMLElement;

		// Add resizing class
		gridItem.classList.add('resizing');

		function onMouseMove(moveEvent: MouseEvent) {
			const deltaX = moveEvent.pageX - startX;
			const deltaY = moveEvent.pageY - startY;
			const gridRect = gridItem.getBoundingClientRect();

			if (direction === 'right') {
				const cellWidth = gridRect.width / item.columnSpan;
				const newSpan = Math.max(1, Math.round(initialColumnSpan + deltaX / cellWidth));
				if (newSpan !== item.columnSpan) {
					gridItems = gridItems.map((gi) =>
						gi.id === item.id ? { ...gi, columnSpan: newSpan } : gi
					);
					updateGridTemplate();
				}
			} else {
				const cellHeight = gridRect.height / item.rowSpan;
				const newSpan = Math.max(1, Math.round(initialRowSpan + deltaY / cellHeight));
				if (newSpan !== item.rowSpan) {
					gridItems = gridItems.map((gi) => (gi.id === item.id ? { ...gi, rowSpan: newSpan } : gi));
					updateGridTemplate();
				}
			}
		}

		function onMouseUp() {
			window.removeEventListener('mousemove', onMouseMove);
			window.removeEventListener('mouseup', onMouseUp);
			// Remove resizing class
			gridItem.classList.remove('resizing');
		}

		window.addEventListener('mousemove', onMouseMove);
		window.addEventListener('mouseup', onMouseUp);
	}

	function updateGridTemplate() {
		// Calculate grid template based on item spans
		const maxColumns = Math.max(
			...gridItems.map((item) => {
				const areaMatch = item.area?.match(/(\d+)/);
				return areaMatch ? parseInt(areaMatch[1]) + item.columnSpan : item.columnSpan;
			})
		);

		gridProperties.gridTemplateColumns = `repeat(${maxColumns}, 1fr)`;
		updatePreviewStyles();
	}

	function applyGridTemplateAreas() {
		// Parse the template into a 2D array of area names
		const areaLines = pendingGridTemplateAreas
			.split('\n')
			.map((line) => line.replace(/['\"]+/g, '').trim())
			.filter((line) => line.length > 0);
		const areaGrid: string[][] = areaLines.map((line) => line.split(/\s+/));

		// Generate unique area names for each cell
		const areaCount: Record<string, number> = {};
		const uniqueAreaGrid: string[][] = areaGrid.map((row) =>
			row.map((cell) => {
				if (cell === '.') return '.';
				areaCount[cell] = (areaCount[cell] || 0) + 1;
				return `${cell}-${areaCount[cell]}`;
			})
		);

		// Flatten unique area names for item creation
		const uniqueAreas: string[] = uniqueAreaGrid.flat().filter((cell) => cell !== '.');

		// Build new gridItems array, one per cell
		const palette = [
			'#8B5CF6',
			'#2DD4BF',
			'linear-gradient(45deg, #8B5CF6, #2DD4BF)',
			'linear-gradient(45deg, #3B82F6, #2DD4BF)',
			'linear-gradient(45deg, #EC4899, #8B5CF6)'
		];
		const newGridItems: GridItem[] = uniqueAreas.map((area, idx) => ({
			id: idx + 1,
			content: area,
			area,
			background: palette[idx % palette.length],
			columnSpan: 1,
			rowSpan: 1
		}));
		gridItems = newGridItems;
		nextId = gridItems.length + 1;

		// Rebuild the template string with unique area names
		const newTemplate = uniqueAreaGrid.map((row) => `'${row.join(' ')}'`).join('\n');
		gridProperties.gridTemplateAreas = newTemplate;
		pendingGridTemplateAreas = newTemplate;

		updatePreviewStyles();
	}
</script>

<div class="container" bind:this={container}>
	{#if mounted}
		<div class="left-panel">
			<div class="sidebar-actions">
				<button class="action-btn add-btn" on:click={addGridItem} title="Add a new grid item"
					>Add Grid Item</button
				>
				<button
					class="action-btn clear-btn"
					on:click={clearAll}
					title="Clear all grid items and template">Clear All</button
				>
			</div>
			<div class="toolbox">
				<h2>Grid Properties Toolbox</h2>

				<section class="property-group">
					<h3>Grid Template</h3>

					<div class="property-control">
						<Tooltip text={propertyExplanations.columns}>
							<label>Columns:</label>
						</Tooltip>
						<div class="input-group">
							<input
								type="number"
								bind:value={columnCount}
								min="1"
								max="12"
								on:change={updateColumns}
							/>
							<input
								type="number"
								bind:value={columnSize}
								min="1"
								max="100"
								on:change={updateColumns}
							/>
							<select bind:value={columnUnit} on:change={updateColumns}>
								<option value="fr">fr</option>
								<option value="px">px</option>
								<option value="%">%</option>
							</select>
						</div>
					</div>

					<div class="property-control">
						<Tooltip text={propertyExplanations.templateAreas}>
							<label>Grid Template Areas:</label>
						</Tooltip>
						<textarea
							bind:value={pendingGridTemplateAreas}
							placeholder="Example:\n'header header header'\n'sidebar main main'\n'footer footer footer'"
							rows="4"
						></textarea>
						<button style="margin-top: 0.5rem;" on:click={applyGridTemplateAreas}>Apply</button>
					</div>
				</section>

				<section class="property-group">
					<h3>Gap</h3>
					<div class="property-control">
						<Tooltip text={propertyExplanations.columnGap}>
							<label>Column Gap:</label>
						</Tooltip>
						<div class="input-group">
							<input
								type="number"
								bind:value={columnGapSize}
								min="0"
								max="100"
								on:input={updateGaps}
							/>
							<select bind:value={columnGapUnit} on:change={updateGaps}>
								<option value="px">px</option>
								<option value="rem">rem</option>
								<option value="%">%</option>
								<option value="em">em</option>
							</select>
						</div>
					</div>
					<div class="property-control">
						<Tooltip text={propertyExplanations.rowGap}>
							<label>Row Gap:</label>
						</Tooltip>
						<div class="input-group">
							<input
								type="number"
								bind:value={rowGapSize}
								min="0"
								max="100"
								on:input={updateGaps}
							/>
							<select bind:value={rowGapUnit} on:change={updateGaps}>
								<option value="px">px</option>
								<option value="rem">rem</option>
								<option value="%">%</option>
								<option value="em">em</option>
							</select>
						</div>
					</div>
				</section>

				<section class="property-group">
					<h3>Auto Grid</h3>
					<div class="property-control">
						<Tooltip text={propertyExplanations.autoFlow}>
							<label>Grid Auto Flow:</label>
						</Tooltip>
						<select bind:value={gridProperties.gridAutoFlow}>
							{#each propertyOptions.gridAutoFlow as option}
								<option value={option}>{option}</option>
							{/each}
						</select>
					</div>
					<div class="property-control">
						<Tooltip text={propertyExplanations.autoColumns}>
							<label>Grid Auto Columns:</label>
						</Tooltip>
						<input type="text" bind:value={gridProperties.gridAutoColumns} />
					</div>
					<div class="property-control">
						<Tooltip text={propertyExplanations.autoRows}>
							<label>Grid Auto Rows:</label>
						</Tooltip>
						<input type="text" bind:value={gridProperties.gridAutoRows} />
					</div>
				</section>

				<section class="property-group">
					<h3>Alignment (Container)</h3>
					<div class="property-control">
						<Tooltip text={propertyExplanations.justifyContent}>
							<label>Justify Content:</label>
						</Tooltip>
						<select bind:value={gridProperties.justifyContent}>
							{#each propertyOptions.justifyContent as option}
								<option value={option}>{option}</option>
							{/each}
						</select>
					</div>
					<div class="property-control">
						<Tooltip text={propertyExplanations.alignContent}>
							<label>Align Content:</label>
						</Tooltip>
						<select bind:value={gridProperties.alignContent}>
							{#each propertyOptions.alignContent as option}
								<option value={option}>{option}</option>
							{/each}
						</select>
					</div>
					<div class="property-control">
						<Tooltip text={propertyExplanations.justifyItems}>
							<label>Justify Items:</label>
						</Tooltip>
						<select bind:value={gridProperties.justifyItems}>
							{#each propertyOptions.justifyItems as option}
								<option value={option}>{option}</option>
							{/each}
						</select>
					</div>
					<div class="property-control">
						<Tooltip text={propertyExplanations.alignItems}>
							<label>Align Items:</label>
						</Tooltip>
						<select bind:value={gridProperties.alignItems}>
							{#each propertyOptions.alignItems as option}
								<option value={option}>{option}</option>
							{/each}
						</select>
					</div>
				</section>

				<section class="property-group">
					<h3>Preset Layouts</h3>
					<div class="select-wrapper">
						<select
							class="layout-select"
							bind:value={currentLayout}
							on:change={() => applyPresetLayout(currentLayout)}
						>
							<option value="simple">Simple Layout</option>
							<option value="dashboard">Dashboard Layout</option>
							<option value="magazine">Magazine Layout</option>
							<option value="megaComplex">Mega Complex Layout (40 items!)</option>
						</select>
						<svg
							class="select-arrow"
							xmlns="http://www.w3.org/2000/svg"
							width="24"
							height="24"
							viewBox="0 0 24 24"
							fill="none"
							stroke="currentColor"
							stroke-width="2"
							stroke-linecap="round"
							stroke-linejoin="round"
						>
							<polyline points="6 9 12 15 18 9"></polyline>
						</svg>
					</div>
				</section>
			</div>
		</div>

		<div class="resizer" on:mousedown={startResize} class:active={isResizing}></div>

		<div class="right-panel">
			<div class="preview-section">
				<div class="preview-container" on:dragover|preventDefault>
					{#each gridItems as item (item.id)}
						<div
							class="grid-item"
							style:grid-area={item.area || 'auto'}
							style:background={item.background || '#e3f2fd'}
							style:grid-column={`span ${item.columnSpan}`}
							style:grid-row={`span ${item.rowSpan}`}
							data-id={item.id}
						>
							<div class="item-content">
								{item.content}
							</div>
							<button
								class="delete-button"
								on:click={() => deleteGridItem(item.id)}
								title="Delete item"
							>
								<svg
									xmlns="http://www.w3.org/2000/svg"
									width="16"
									height="16"
									viewBox="0 0 24 24"
									fill="none"
									stroke="currentColor"
									stroke-width="2"
									stroke-linecap="round"
									stroke-linejoin="round"
								>
									<path d="M3 6h18"></path>
									<path d="M19 6v14c0 1-1 2-2 2H7c-1 0-2-1-2-2V6"></path>
									<path d="M8 6V4c0-1 1-2 2-2h4c1 0 2 1 2 2v2"></path>
								</svg>
							</button>
							<div
								class="resize-handle resize-right"
								on:mousedown={(e) => handleResize(e, item, 'right')}
								title="Drag to resize horizontally"
							></div>
							<div
								class="resize-handle resize-bottom"
								on:mousedown={(e) => handleResize(e, item, 'bottom')}
								title="Drag to resize vertically"
							></div>
						</div>
					{/each}
				</div>
			</div>

			<div class="code-preview">
				<div class="code-header">
					<h3>CSS Code Preview</h3>
					<button class="copy-button" on:click={copyToClipboard}>Copy to Clipboard</button>
				</div>
				<pre class="language-css"><code bind:this={codeElement} class="language-css"
						>{codePreview}</code
					></pre>
			</div>
		</div>
	{/if}
</div>

<style>
	.container {
		display: grid;
		grid-template-columns: 300px 8px 1fr;
		height: 100vh;
		width: 100%;
		overflow: hidden;
	}

	.left-panel {
		background-color: #f5f5f5;
		padding: 1rem;
		overflow-y: auto;
	}

	.toolbox {
		display: flex;
		flex-direction: column;
		gap: 1rem;
	}

	.property-group {
		background-color: white;
		padding: 1rem;
		border-radius: 4px;
		box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
	}

	.property-group h3 {
		margin: 0 0 1rem 0;
		color: #2196f3;
		font-size: 1.1rem;
	}

	.property-control {
		margin-bottom: 1rem;
	}

	.property-control label {
		display: block;
		margin-bottom: 0.5rem;
		font-size: 0.9rem;
		color: #666;
	}

	.property-control input,
	.property-control select,
	.property-control textarea {
		width: 100%;
		padding: 0.5rem;
		border: 1px solid #ddd;
		border-radius: 4px;
		font-size: 0.9rem;
	}

	.property-control textarea {
		font-family: monospace;
		resize: vertical;
	}

	.input-group {
		display: grid;
		grid-template-columns: 1fr 1fr 1fr;
		gap: 0.5rem;
	}

	.toolbox button {
		padding: 0.5rem 1rem;
		background-color: #4caf50;
		color: white;
		border: none;
		border-radius: 4px;
		cursor: pointer;
		transition: background-color 0.2s;
	}

	.toolbox button:hover {
		background-color: #45a049;
	}

	.resizer {
		background-color: #e0e0e0;
		cursor: col-resize;
		transition: background-color 0.2s;
	}

	.resizer:hover,
	.resizer.active {
		background-color: #2196f3;
	}

	.right-panel {
		background-color: white;
		padding: 1rem;
		overflow-y: auto;
		display: flex;
		flex-direction: column;
		gap: 1rem;
	}

	.preview-section {
		flex: 1;
	}

	.code-preview {
		background-color: #1e1e1e;
		border-radius: 4px;
		padding: 1rem;
		font-family: 'Consolas', 'Monaco', 'Courier New', monospace;
		max-height: 300px;
		overflow-y: auto;
	}

	.code-header {
		display: flex;
		justify-content: space-between;
		align-items: center;
		margin-bottom: 1rem;
	}

	.code-header h3 {
		color: #d4d4d4;
		margin: 0;
	}

	.copy-button {
		background-color: #2196f3;
		color: white;
		border: none;
		border-radius: 4px;
		padding: 0.5rem 1rem;
		cursor: pointer;
		font-size: 0.9rem;
		transition: background-color 0.2s;
	}

	.copy-button:hover {
		background-color: #1976d2;
	}

	.code-preview pre {
		margin: 0;
		padding: 0;
		background: transparent;
	}

	.code-preview code {
		display: block;
		line-height: 1.5;
		font-size: 0.9rem;
		tab-size: 2;
	}

	/* Override some Prism styles to match our theme */
	:global(.code-preview .token.property) {
		color: #9cdcfe;
	}

	:global(.code-preview .token.selector) {
		color: #d7ba7d;
	}

	:global(.code-preview .token.punctuation) {
		color: #808080;
	}

	:global(.code-preview .token.value) {
		color: #ce9178;
	}

	.preview-container {
		display: grid;
		padding: 1rem;
		background-color: #fafafa;
		border-radius: 4px;
		min-height: 300px;
		border: 1px solid #e0e0e0;
		gap: 1rem;
		position: relative; /* Ensure container is positioned */
	}

	.grid-item {
		padding: 1rem;
		border-radius: 4px;
		display: flex;
		align-items: center;
		justify-content: center;
		color: white;
		font-weight: 500;
		text-shadow: 0 1px 2px rgba(0, 0, 0, 0.1);
		box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
		position: relative;
		transition:
			transform 0.2s,
			box-shadow 0.2s;
		user-select: none;
		min-height: 80px;
		min-width: 80px;
	}

	.grid-item:hover {
		transform: translateY(-2px);
		box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
	}

	.item-content {
		flex: 1;
		text-align: center;
	}

	.delete-button {
		position: absolute;
		top: 8px;
		right: 8px;
		background: rgba(255, 255, 255, 0.2);
		border: none;
		border-radius: 4px;
		width: 28px;
		height: 28px;
		display: flex;
		align-items: center;
		justify-content: center;
		cursor: pointer;
		opacity: 0;
		transition:
			opacity 0.2s,
			background-color 0.2s;
		color: white;
		padding: 0;
	}

	.delete-button:hover {
		background: rgba(255, 255, 255, 0.3);
	}

	.grid-item:hover .delete-button {
		opacity: 1;
	}

	/* Ensure the SVG icon inherits the button's color */
	.delete-button svg {
		width: 16px;
		height: 16px;
	}

	.select-wrapper {
		position: relative;
		width: 100%;
	}

	.layout-select {
		width: 100%;
		padding: 0.75rem;
		padding-right: 2.5rem;
		background-color: #8b5cf6;
		color: white;
		border: none;
		border-radius: 4px;
		cursor: pointer;
		font-weight: 500;
		appearance: none;
		-webkit-appearance: none;
		transition: background-color 0.2s;
	}

	.layout-select:hover {
		background-color: #7c3aed;
	}

	.layout-select:focus {
		outline: 2px solid #c4b5fd;
		outline-offset: 2px;
	}

	.select-arrow {
		position: absolute;
		right: 0.75rem;
		top: 50%;
		transform: translateY(-50%);
		color: white;
		pointer-events: none;
		width: 20px;
		height: 20px;
	}

	/* Style the dropdown options (works in modern browsers) */
	.layout-select option {
		background-color: white;
		color: #1f2937;
		padding: 0.5rem;
	}

	.resize-handle {
		position: absolute;
		background-color: rgba(255, 255, 255, 0.2);
		transition: background-color 0.2s;
		z-index: 2;
	}

	.resize-handle::after {
		content: '';
		position: absolute;
		background-color: inherit;
		opacity: 0;
		transition: opacity 0.2s;
	}

	.resize-right {
		right: 0;
		top: 0;
		width: 4px;
		height: 100%;
		cursor: col-resize;
	}

	.resize-right::after {
		top: 50%;
		right: 0;
		transform: translateY(-50%);
		width: 12px;
		height: 40px;
		border-radius: 6px;
	}

	.resize-bottom {
		bottom: 0;
		left: 0;
		width: 100%;
		height: 4px;
		cursor: row-resize;
	}

	.resize-bottom::after {
		left: 50%;
		bottom: 0;
		transform: translateX(-50%);
		width: 40px;
		height: 12px;
		border-radius: 6px;
	}

	.grid-item:hover .resize-handle::after {
		opacity: 1;
	}

	.sidebar-actions {
		display: flex;
		gap: 0.5rem;
		justify-content: center;
		margin-bottom: 1rem;
	}
	.action-btn {
		padding: 0.25rem 0.75rem;
		font-size: 0.85rem;
		border: none;
		border-radius: 4px;
		cursor: pointer;
		transition: background 0.2s;
		min-width: 110px;
	}
	.add-btn {
		background: #4caf50;
		color: white;
	}
	.add-btn:hover {
		background: #388e3c;
	}
	.clear-btn {
		background: #f44336;
		color: white;
	}
	.clear-btn:hover {
		background: #d32f2f;
	}
</style>
