<template>
  <div>
    <b-container>
      <b-row>
        <b-col>
          <h1 class="font-weight-bolder">GCode Looper</h1>
          <img src="@/assets/gcode-looper-logo.svg"/>
          <p class="mt-3 mb-4">Upload a GCode file to print and eject multiple copies of your 3D print.</p>
          <input type="file" accept=".gcode" @change="onFileChange" ref="fileInput" style="display:none" />
          <div class="dropzone" @click="browseFile" @drop.prevent="onDrop" @dragover.prevent>
            <span class="font-weight-bolder">Drag & Drop GCode File Here</span>
            <br />
            <span class="browse-link">Browse to Upload</span>
          </div>
          <p class="my-3"><small>Currently only supports single object gcode files sliced with latest PrusaSlicer</small></p>
        </b-col>
      </b-row>
      <hr v-if="gcodeText" class="my-3"/>
      <b-row>
        <b-col md="6" class="mb-3 mt-2">
          
          <img class="w-100" :src="thumbnailSrc" alt="Thumbnail preview" v-if="thumbnailSrc" />
          <h3 v-if="fileName" class="filename-display">{{ fileName }}</h3>

          <b-table-simple v-if="fileName" class="mt-3" responsive striped bordered small>
          <b-tbody>
            <!-- 3D Specifications Header -->
            <b-tr class="text-center bg-info text-white table-header"><b-td :colspan="2">File Specifications</b-td></b-tr>
            <!-- Height Row -->
            <b-tr><b-td>Bed Position [X,Y]</b-td><b-td>{{ ((gcodeData.bounds.maxX + gcodeData.bounds.minX) / 2).toFixed(2)}}, {{ ((gcodeData.bounds.maxY + gcodeData.bounds.minY) / 2).toFixed(2)}}</b-td></b-tr>
            <b-tr><b-td>Height</b-td><b-td>{{ (gcodeData.height).toFixed(2) }}</b-td></b-tr>
            <b-tr><b-td>Width</b-td><b-td>{{ ((gcodeData.bounds.maxX - gcodeData.bounds.minX) + gcodeData.extrusionWidth).toFixed(2)}}</b-td></b-tr>
            <b-tr><b-td>Length</b-td><b-td>{{ ((gcodeData.bounds.maxY - gcodeData.bounds.minY) + gcodeData.extrusionWidth).toFixed(2)}}</b-td></b-tr>
            <b-tr><b-td>X Bounds [min,max]</b-td><b-td>{{ (gcodeData.bounds.minX - (gcodeData.extrusionWidth / 2)).toFixed(2)}}, {{ (gcodeData.bounds.maxX + (gcodeData.extrusionWidth / 2)).toFixed(2)}}</b-td></b-tr>
            <b-tr><b-td>Y Bounds [min,max]</b-td><b-td>{{ (gcodeData.bounds.minY - (gcodeData.extrusionWidth / 2)).toFixed(2)}}, {{ (gcodeData.bounds.maxY + (gcodeData.extrusionWidth / 2)).toFixed(2)}}</b-td></b-tr>
            
          </b-tbody>
        </b-table-simple>
          
          <div v-if="gcodeText" class="gcode-window mt-4">
            <pre>{{ gcodeText }}</pre>
          </div>
          <small v-if="gcodeText">Original GCode</small>
        </b-col>
        <b-col md="6" class="mb-3">
          <!-- Input Form -->
          <b-form v-if="gcodeText" @submit.prevent>

            <b-table-simple v-if="fileName" class="mt-2" responsive striped bordered small>
              <b-tbody>
                <!-- Estimated Totals Header -->
                <b-tr class="text-center bg-info text-white table-header"><b-td :colspan="2">Estimated Totals</b-td></b-tr>
                <b-tr v-b-tooltip.hover title="Days : Hours : Minutes : Seconds"><b-td>Print Time</b-td><b-td>{{ gcodeData.totalPrintTime }}</b-td></b-tr>
                <b-tr><b-td>Grams of Filament</b-td><b-td>{{ gcodeData.totalGramsOfFilament }}</b-td></b-tr>
                <b-tr><b-td>Cost of Filament</b-td><b-td>{{ gcodeData.totalCostOfFilament }}</b-td></b-tr>
              </b-tbody>
            </b-table-simple>

            <b-input-group class="mb-3">
              <b-input-group-text v-b-tooltip.hover title="The number of copies of your 3D print.">Copies:</b-input-group-text>
              <b-form-input type="number" v-model="form.copies" min="1" required></b-form-input>
            </b-input-group>

            <b-input-group class="mb-3">
              <b-input-group-text v-b-tooltip.hover title="The side of your 3D printer you want the parts to be ejected from.">Ejection Side:</b-input-group-text>
              <b-form-select v-model="form.ejectionSide" :options="['Left', 'Right', 'Front', 'Back']" required></b-form-select>
            </b-input-group>

            <!-- Conditionally rendered input fields -->
            <b-input-group v-if="form.ejectionSide === 'Back'" class="mb-3">
              <b-input-group-text v-b-tooltip.hover title="The length in millimeters of the bed of your 3D printer. This will be used for the ejection of each print.">Bed Length (mm):</b-input-group-text>
              <b-form-input type="number" v-model="form.bedLength" step="0.01" required></b-form-input>
            </b-input-group>
            <b-input-group v-if="form.ejectionSide === 'Right'" class="mb-3">
              <b-input-group-text v-b-tooltip.hover title="The length in millimeters of the bed of your 3D printer. This will be used for the ejection of each print.">Bed Width (mm):</b-input-group-text>
              <b-form-input type="number" v-model="form.bedWidth" step="0.01" required></b-form-input>
            </b-input-group>
            
            <b-input-group class="mb-3">
              <b-input-group-text v-b-tooltip.hover title="How far the nozzle of your 3D printer will move away from the bounds of your part when it is ejected.">Print Head Offset (mm):</b-input-group-text>
              <b-form-input type="number" v-model="form.ejectionOffset" step="0.01" required></b-form-input>
            </b-input-group>

            <b-input-group class="mb-3">
              <b-input-group-text v-b-tooltip.hover title="How far the nozzle of your 3D printer will be raised off the print bed while the part is ejected">Push Height (mm):</b-input-group-text>
              <b-form-input type="number" v-model="form.pushHeight" step="0.01" required></b-form-input>
            </b-input-group>


          </b-form>
          <b-button @click="downloadFile" variant="success" v-if="gcodeText">Download New GCode</b-button>
        </b-col>
        
      </b-row>
      <b-row class="my-4">
        <b-col>
          <p>Finding this web app useful? Consider donating</p>
          <a href="https://github.com/sponsors/CurlyTaleGames" target="_blank" class="btn btn-light mx-2">
            <img src="@/assets/github.svg" alt="Github Sponsors" height="22px"/> GitHub Sponsors
          </a>
          <a href="https://www.paypal.com/donate/?hosted_button_id=L4GAK93DRELFW" target="_blank" class="btn btn-light mx-2">
            <img src="@/assets/PayPal.svg" alt="PayPal" height="22px"/>
          </a>
        </b-col>
      </b-row>
    </b-container>
  </div>
</template>

<script>

export default {
  data() {
    return {
      gcodeText: '',
      thumbnailSrc: '',
      fileName: '',
      newFileName: '',
      form: {
        copies: 10,
        ejectionSide: 'Front',
        pushHeight: 20.0,
        ejectionOffset: 40.0,
        bedLength: null,
        bedWidth: null
      },
      gcodeData:{
        height: 0,
        minX: 0,
        maxX: 0,
        minY: 0,
        maxY: 0,
        extrusionWidth: 0,
        bounds:{
          minX: 0,
          maxX: 0,
          minY: 0,
          maxY: 0
        },
        printTimeSeconds: 0,
        totalPrintTime: 0,
        totalGramsOfFilament: 0,
        totalCostOfFilament: 0
      }
    };
  },
  methods: {
    browseFile() {
      this.$refs.fileInput.click();
    },
    replaceFile() {
      this.gcodeText = '';
      this.thumbnailSrc = '';
      this.fileName = '';
      this.$refs.fileInput.click();
    },
    onFileChange(e) {
      const files = e.target.files || e.dataTransfer.files;
      if (!files.length) return;

      // Check the file extension
      if (files[0].name.split('.').pop().toLowerCase() !== 'gcode') {
        alert('Invalid file type. Please upload a .gcode file.');
        return;
      }

      this.fileName = files[0].name;
      this.createFile(files[0]);
    },
    onDrop(e) {
      // Get the dropped files
      const files = e.dataTransfer.files;
      if (!files.length) return;

      // Check the file extension
      if (files[0].name.split('.').pop().toLowerCase() !== 'gcode') {
        alert('Invalid file type. Please upload a .gcode file.');
        return;
      }

      this.onFileChange(e);
    },
    createFile(file) {
      const reader = new FileReader();
      reader.onload = (e) => {
        this.gcodeText = e.target.result;
        this.thumbnailSrc = this.extractLargestThumbnail(this.gcodeText);
        this.gcodeData.extrusionWidth = this.parseExtrusionWidth(this.gcodeText);
        this.gcodeData.height = this.parseZHeight(this.gcodeText);
        this.gcodeData.printTimeSeconds = this.parseAndConvertTime(this.gcodeText);
        this.gcodeData.bounds = this.findBounds(this.gcodeText);
      };
      reader.readAsText(file);
    },
    extractLargestThumbnail(gcode) {
      console.log("Extracting Text");
      console.log(gcode);
      const matches = gcode.match(/; thumbnail begin (\d+)x(\d+) (\d+)([\s\S+]*?); thumbnail end/g);
      if (!matches) return '';

      let largestThumbnail = '';
      let largestResolution = 0;

      matches.forEach((match) => {
        console.log(match);
        const resolutionMatch = match.match(/; thumbnail begin (\d+)x(\d+) (\d+)/);
        if (resolutionMatch) {
          const width = parseInt(resolutionMatch[1], 10);
          const height = parseInt(resolutionMatch[2], 10);
          const resolution = width * height;
          if (resolution > largestResolution) {
            largestResolution = resolution;
            // Extract the base64 thumbnail data
            let base64String = match.replace(/^.*\n|\n|;\s*/g, '');
            base64String = base64String.replace(/=thumbnail end/g, '=');
            largestThumbnail = 'data:image/png;base64, ' + base64String;
          }
        }
      });
      return largestThumbnail;
    },
    downloadFile() {
      const blob = new Blob([this.gcodeText], { type: 'text/plain' });
      const link = document.createElement('a');
      link.href = URL.createObjectURL(blob);
      link.download = this.createNewFilename(this.fileName, this.form.copies, this.gcodeData.printTimeSeconds) || 'downloaded.gcode'; // Use stored fileName for download
      link.click();
    },
    parseZHeight(gcode){
      let highestZ = 0;
      // Split the G-code text into lines
      const lines = gcode.split('\n');
      // Iterate over each line
      lines.forEach(line => {
        // Check if the line starts with ;Z:
        if (line.startsWith(';Z:')) {
          // Parse the Z height value from the line
          const zHeight = parseFloat(line.split(';Z:')[1]);
          // Update highestZ if this zHeight is greater than the current highestZ
          if (zHeight > highestZ) {
            highestZ = zHeight;
          }
        }
      });
      return highestZ;
    },
    parseAndConvertTime(gcode) {
      // This regex captures optional days, hours, minutes, and seconds parts
      const timeMatch = gcode.match(/estimated printing time.*?(?:(\d+)d )?(?:(\d+)h )?(\d+)m (\d+)s/);

      // Check if the match is found and has the expected groups
      if (timeMatch) {
        // Convert the time to seconds
        const days = parseInt(timeMatch[1] || 0, 10); // Default to 0 if days part is not present
        const hours = parseInt(timeMatch[2] || 0, 10); // Default to 0 if hours part is not present
        const minutes = parseInt(timeMatch[3], 10);
        const seconds = parseInt(timeMatch[4], 10);
        const totalSeconds = (days * 86400) + (hours * 3600) + (minutes * 60) + seconds; // 1 day = 86400 seconds

        // Return the total time in seconds
        return totalSeconds;
      } else {
        // Return null or some default value if the time cannot be parsed
        console.warn('Could not parse estimated printing time from G-code');
        return null;
      }
    },
    parseGramsOfFilament(gcode) {
      // This regex captures the grams of filament used
      const gramsMatch = gcode.match(/total filament used \[g\] = ([0-9.]+)/);

      // Check if the match is found and has the expected group
      if (gramsMatch) {
        // Convert the grams to a floating-point number
        const grams = parseFloat(gramsMatch[1]);

        // Return the total grams of filament used
        return grams;
      } else {
        // Return null or some default value if the grams cannot be parsed
        console.warn('Could not parse total grams of filament used from G-code');
        return null;
      }
    },
    parseExtrusionWidth(gcode) {
      // This regex captures the extrusion width, accommodating for variations in format
      const extrusionMatch = gcode.match(/; extrusion_width = (\d+(\.\d+)?)/);

      // Check if the match is found and has the expected group
      if (extrusionMatch) {
        // Convert the width to a floating-point number
        const extrusion = parseFloat(extrusionMatch[1]);

        // Return the extracted extrusion width
        return extrusion;
      } else {
        // Return null or some default value if the width cannot be parsed
        console.warn('Could not parse extrusion width used from G-code');
        return null;
      }
    },
    parseFilamentCost(gcodeText) {
      const match = gcodeText.match(/; total filament cost = (\d+(\.\d+)?)/); // Regex to match cost line and capture cost value
      if (match && match[1]) { // If match is found and the captured group is not undefined
        return parseFloat(match[1]); // Convert the captured group (cost value) to a float and return it
      } else {
        console.error('Filament cost not found in gcode text.'); // Log an error if the cost line isn't found
        return 0; // Return a default value (like 0) if no match is found
      }
    },
    convertSecondsToTimeFormat(seconds) {
      const days = Math.floor(seconds / 86400);
      const hrs = Math.floor((seconds % 86400) / 3600);
      const mins = Math.floor((seconds % 3600) / 60);
      const secs = seconds % 60;

      const timeString = [days, hrs, mins, secs]
        .map(v => (v < 10 ? '0' : '') + v)
        .join(':');

      return timeString;
    },
    convertSecondsToDayHourMinuteFormat(seconds) {
      const days = Math.floor(seconds / 86400);
      seconds -= days * 86400;

      const hours = Math.floor(seconds / 3600);
      seconds -= hours * 3600;

      const minutes = Math.floor(seconds / 60);

      // Construct the formatted string
      let timeString = '';
      if (days > 0) timeString += `${days}d`;
      if (hours > 0) timeString += `${hours}h`;
      if (minutes > 0) timeString += `${minutes}m`;

      // Handle the case where the input seconds are zero or the calculations result in zero
      if (!timeString) timeString = '0m';

      return timeString;
    },
    createNewFilename(filename, copies, totalTime) {
      const parts = filename.split('_');
      if (parts.length >= 3) {
        const firstPart = parts[0];
        const middlePart = parts.slice(1, -1).join('_');
        return firstPart + "-x" + copies + "_" + middlePart + "_" + this.convertSecondsToDayHourMinuteFormat(copies * totalTime) + ".gcode";
      }
      console.warn('Not enough underscores found in filename');
      return [filename];
    },
    findBounds(gcode) {
      let recording = false;
      let minX = Infinity;
      let maxX = -Infinity;
      let minY = Infinity;
      let maxY = -Infinity;

      const lines = gcode.split('\n');
      for (const line of lines) {
        if (line.includes(';TYPE:External perimeter')) {
          recording = true;
          continue;
        }

        if (recording && line.includes(';WIPE_START')) {
          recording = false;
        }

        if (recording && line.startsWith('G1')) {
          const coords = this.extractXYCoords(line);
          if (coords) {
            const { x, y } = coords;
            minX = Math.min(minX, x);
            maxX = Math.max(maxX, x);
            minY = Math.min(minY, y);
            maxY = Math.max(maxY, y);
          }
        }
      }
      
      return {
        minX: minX === Infinity ? null : minX,
        maxX: maxX === -Infinity ? null : maxX,
        minY: minY === Infinity ? null : minY,
        maxY: maxY === -Infinity ? null : maxY,
      };
    },
    extractXYCoords(line) {
      const xMatch = line.match(/X([-\d.]+)/);
      const yMatch = line.match(/Y([-\d.]+)/);
      if (!xMatch || !yMatch) return null;
      return {
        x: parseFloat(xMatch[1]),
        y: parseFloat(yMatch[1]),
      };
    }
  },
  computed: {
    totalPrintTimeSeconds() {
      return this.gcodeData.printTimeSeconds * this.form.copies;
    },
    totalPrintTime() {
      return this.convertSecondsToTimeFormat(this.totalPrintTimeSeconds);
    },
    totalGramsOfFilament() {
      return (this.parseGramsOfFilament(this.gcodeText) * this.form.copies).toFixed(2);
    },
    totalCostOfFilament() {
      return "$" + (this.parseFilamentCost(this.gcodeText) * this.form.copies).toFixed(2);
    },
  },
  watch: {
    totalPrintTime(newValue) {
      this.gcodeData.totalPrintTime = newValue;
    },
    totalGramsOfFilament(newValue) {
      this.gcodeData.totalGramsOfFilament = newValue;
    },
    totalCostOfFilament(newValue) {
      this.gcodeData.totalCostOfFilament = newValue;
    },
  },
};
</script>

<style>
.dropzone {
  border: 2px dashed #6aa961;
  padding: 20px;
  cursor: pointer;
  text-align: center;
  background-color: rgba(106, 169, 97, 12%);
}

.dropzone:hover {
  background-color: rgba(106, 169, 97, 25%);
}

.dropzone span{
  font-weight: 700 !important;
}

.browse-link {
  display: inline-block;
  margin-top: 10px;
  color: #6aa961;
  cursor: pointer;
  text-decoration: underline;
}

.gcode-window {
  border: 1px solid #ccc;
  padding: 10px;
  height: 200px;
  overflow-y: scroll;
}

pre{
  text-align: left;
}

.filename-display {
  margin: 10px 0;
  font-weight: bold;
  font-size: calc(0.5vw + 0.5vh + .5vmin)
}

h1{
  font-weight: 700 !important;
}

small{
  opacity: 50%;
  margin: 30px 0;
}

/* fix for missing tooltip arrow */
.tooltip .arrow {
    position: absolute;
    display: block;
    width: 0.8rem;
    height: 0.4rem;
}

.bs-tooltip-top .arrow {
    bottom: -6px;
}

.tooltip .arrow::before {
    content: "";
    position: absolute;
    border-color: transparent;
    border-style: solid;
}

.bs-tooltip-top .arrow::before {
    border-width: 0.4rem 0.4rem 0;
    border-top-color: #000000;
} 

.table-header{
  font-weight: 700 !important;
}

</style>
