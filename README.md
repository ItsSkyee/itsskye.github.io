# itsskye.github.io

<!DOCTYPE html>
<title>3v.fi - Discord Timestamp Generator</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<style>
html,body{margin:0;padding:0;background:#161620;color:#eee;font-family:sans-serif;}
.container{display:grid;grid-template-columns:1fr max-content;width:50rem;margin:0 auto;border-right:1px solid #666;border-left:1px solid #666;padding:1rem;}
.container>*{margin:0.2rem;}
#current{grid-column:span 2;}
input,select,button{background:#383842;color:#fff;border:1px solid #666;}
#code{font-family:monospace}

@media (max-width: 50rem) {
	div.container{display:flex;flex-direction:column;width:90%}
	div.container>*{margin:1rem 0.2rem}
}
</style>
<div class="container">
	<p>Get a dynamic date-time display in your Discord messages. Choose your time and copy the code below.</p>
</div>
<div class="container">
	<span>Date</span><input type="date" id="d">
	<span>Time</span><input type="time" id="hm">
	<span>Type</span>
	<select id="t">
		<option value="t">short time</option>
		<option value="T">long time</option>
		<option value="d">short date</option>
		<option value="D">long date</option>
		<option value="f">long date with short time</option>
		<option value="F">long date with day of week and short time</option>
		<option value="R" selected>relative</option>
	</select>
	<span>Output</span><span id="preview"></span>
	<input type="text" readonly id="code" title="Press Ctrl/Cmd+C to copy"><button id="copy">Copy to clipboard</button>
	<button id="current">Reset to current time</button>
</div>
<script>
const dateInput = document.getElementById('d');
const timeInput = document.getElementById('hm');
const typeInput = document.getElementById('t');
const output = document.getElementById('code');
const copy = document.getElementById('copy');
const current = document.getElementById('current');
const preview = document.getElementById('preview');

dateInput.onchange = updateOutput;
timeInput.onchange = updateOutput;
typeInput.onchange = updateOutput;
output.onmouseover = function() { this.select(); }
copy.onclick = async () => {
	updateOutput();
	try {
		await navigator.clipboard.writeText(output.value);
		alert("Successfully copied");
	} catch (e) {
		alert(e);
	}
}

const onload =_=> {
	const now = new Date();
	dateInput.value = `${now.getFullYear()}-${(now.getMonth()+1).toString().padStart(2, '0')}-${now.getDate().toString().padStart(2, '0')}`;
	timeInput.value = `${now.getHours().toString().padStart(2, '0')}:${now.getMinutes().toString().padStart(2, '0')}`;
	updateOutput();
}
window.onload = onload;
current.onclick = onload;

const typeFormats = {
	't': { timeStyle: 'short' },
	'T': { timeStyle: 'medium' },
	'd': { dateStyle: 'short' },
	'D': { dateStyle: 'long' },
	'f': { dateStyle: 'long', timeStyle: 'short' },
	'F': { dateStyle: 'full', timeStyle: 'short' },
	'R': { style: 'long', numeric: 'auto' },
};

function automaticRelativeDifference(d) {
	const diff = -((new Date().getTime() - d.getTime())/1000)|0;
	const absDiff = Math.abs(diff);
	console.log(diff);
	if (absDiff > 86400*30*10) {
		return { duration: Math.round(diff/(86400*365)), unit: 'years' };
	}
	if (absDiff > 86400*25) {
		return { duration: Math.round(diff/(86400*30)), unit: 'months' };
	}
	if (absDiff > 3600*21) {
		return { duration: Math.round(diff/86400), unit: 'days' };
	}
	if (absDiff > 60*44) {
		return { duration: Math.round(diff/3600), unit: 'hours' };
	}
	if (absDiff > 30) {
		return { duration: Math.round(diff/60), unit: 'minutes' };
	}
	return { duration: diff, unit: 'seconds' };
}

function updateOutput() {
	const selectedDate = new Date(dateInput.valueAsNumber + timeInput.valueAsNumber + new Date().getTimezoneOffset() * 60000);
	console.log(selectedDate);
	const ts = selectedDate.getTime().toString();
	output.value = `<t:${ts.substr(0, ts.length - 3)}:${typeInput.value}>`;

	if (['R'].includes(typeInput.value)) {
		const formatter = new Intl.RelativeTimeFormat(navigator.language || 'en', typeFormats[typeInput.value] || {});
		const format = automaticRelativeDifference(selectedDate);
		preview.textContent = formatter.format(format.duration, format.unit);
	} else {
		const formatter = new Intl.DateTimeFormat(navigator.language || 'en', typeFormats[typeInput.value] || {});
		preview.textContent = formatter.format(selectedDate);
	}
}
</script>
