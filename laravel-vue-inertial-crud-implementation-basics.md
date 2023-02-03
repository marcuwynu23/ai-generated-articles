# Laravel Inertia Vue.js CRUD example

### List Data

- Controller source code for listing Activities.

```php
 public function index()
   {
       $activities = Activity::all();
       return inertia("Activity", [
           "activities" => $activities
       ]);
   }

```

- ### Pages/Activity/Activity.vue
- Vue source code.

```jsx
<script setup>
import Layout from "../../Layouts/Layout.vue";
import { Link, useForm } from "@inertiajs/inertia-vue3";
import { Inertia } from "@inertiajs/inertia";

defineProps({
	activities: Object
})

const form = useForm();

function deleteData(id) {
	form.delete(route('activity.destroy', id));
}

</script>

<template>
	<Layout>
		<h1>Activity Log</h1>
		<section>
			<div class="activity-log-container">
				<table class="table table-responsive">
					<thead class="thead-light">
						<tr>
							<th>#</th>
							<th>Date</th>
							<th>Time</th>
							<th>Accomplishment</th>
							<th>Remarks</th>
							<th>Option</th>
						</tr>
					</thead>
					<tbody>
						<tr v-for="activity in activities">
							<td>{{ activity.id }}</td>
							<td>{{ activity.date }}</td>
							<td>{{ activity.time }}</td>
							<td>{{ activity.accomplishment }}</td>
							<td>{{ activity.remarks }}</td>
							<td>
								<Link :href="route('activity.edit', activity.id)"
									class="d-inline-block btn btn-primary m-2">Edit</Link>
								<form @submit.prevent="deleteData(activity.id)">
									<button class="d-inline-block btn btn-primary m-2">Delete</button>
								</form>
							</td>
						</tr>
					</tbody>
				</table>

			</div>
		</section>
	</Layout>
</template>

```

### Create Form

- #### Controller

```php
public function create()
    {
        return inertia("Activity/ActivityCreateForm");
    }

```

- #### Vue View

```jsx

<script setup>
import Layout from '../../Layouts/Layout.vue';
import { Inertia } from '@inertiajs/inertia';
import { useForm } from '@inertiajs/inertia-vue3';


const form = useForm({
	date: "",
	time: "",
	accomplishment: "",
	remarks: ""
});

function submit() {
	form.post(route('activity.store'));
}
</script>
<template>
	<Layout>
		<h1>Activity Form</h1>
		<section>
			<div class="activity-form-container">
				<form @submit.prevent="submit">
					<div class="form-group">
						<label for="date">Date</label>
						<input type="date"
							class="form-control"
							v-model="form.date"
							required="true">
					</div>
					<div class="form-group">
						<label for="time">Time</label>
						<input type="time"
							class="form-control"
							v-model="form.time"
							required="true">
					</div>
					<div class="form-group">
						<label for="accomplishment">Accomplishment</label>
						<input type="text"
							class="form-control"
							v-model="form.accomplishment"
							required="true">
					</div>
					<div class="form-group">
						<label for="remarks">Remarks</label>
						<input type="text"
							class="form-control"
							v-model="form.remarks">
					</div>
					<br>
					<button type="submit"
						class="btn btn-primary px-2 m-1">Submit</button>
				</form>
			</div>
		</section>
	</Layout>
</template>
```

### StoRe Data From Form

- #### Controller

```php

public function store(Request $request)
    {
        //
        $postData = $request->all();
        DailyLog::create($postData);
        return redirect()->route("activity.index");
    }

```

- #### Vue View

```jsx
<script setup>

import { Inertia } from '@inertiajs/inertia';
import { useForm } from '@inertiajs/inertia-vue3';
import Layout from '../../Layouts/Layout.vue';

const props = defineProps({
	data: {
		type: Object,
		default: () => { }
	}
});





const form = useForm({
	date: props.data.date,
	time: props.data.time,
	accomplishment: props.data.accomplishment,
	remarks: props.data.remarks
});

const update = () => {
	form.patch(route('activity.update', props.data.id));
}
</script>


<template>
	<Layout>
		<h1> Edit Activity Form</h1>
		<section>
			<div class="activity-form-container">
				<form @submit.prevent="update">
					<div class="form-group">
						<label for="date">Date</label>
						<input type="date"
							class="form-control"
							v-model="form.date">
					</div>
					<div class="form-group">
						<label for="time">Time</label>
						<input type="time"
							class="form-control"
							v-model="form.time">
					</div>
					<div class="form-group">
						<label for="accomplishment">Accomplishment</label>
						<input type="text"
							class="form-control"
							v-model="form.accomplishment">
					</div>
					<div class="form-group">
						<label for="remarks">Remarks</label>
						<input type="text"
							class="form-control"
							v-model="form.remarks">
					</div>
					<br>
					<button type="submit"
						class="btn btn-secondary px-2 m-1">Update</button>
				</form>
			</div>
		</section>
	</Layout>
</template>
```

### Update Data

- #### Controller

```php
    public function edit($id)
    {
        return inertia("Activity/ActivityEditForm", [
            "data" => DailyLog::find($id)
        ]);
    }


```

- #### Vue View

```jsx


<script setup>

import { Inertia } from '@inertiajs/inertia';
import { useForm } from '@inertiajs/inertia-vue3';
import Layout from '../../Layouts/Layout.vue';

const props = defineProps({
	data: {
		type: Object,
		default: () => { }
	}
});





const form = useForm({
	date: props.data.date,
	time: props.data.time,
	accomplishment: props.data.accomplishment,
	remarks: props.data.remarks
});

const update = () => {
	form.patch(route('activity.update', props.data.id));
}
</script>


<template>
	<Layout>
		<h1> Edit Activity Form</h1>
		<section>
			<div class="activity-form-container">
				<form @submit.prevent="update">
					<div class="form-group">
						<label for="date">Date</label>
						<input type="date"
							class="form-control"
							v-model="form.date">
					</div>
					<div class="form-group">
						<label for="time">Time</label>
						<input type="time"
							class="form-control"
							v-model="form.time">
					</div>
					<div class="form-group">
						<label for="accomplishment">Accomplishment</label>
						<input type="text"
							class="form-control"
							v-model="form.accomplishment">
					</div>
					<div class="form-group">
						<label for="remarks">Remarks</label>
						<input type="text"
							class="form-control"
							v-model="form.remarks">
					</div>
					<br>
					<button type="submit"
						class="btn btn-secondary px-2 m-1">Update</button>
				</form>
			</div>
		</section>
	</Layout>
</template>
```

### Delete Or Destroy Data

```php
    public function destroy($id)
    {
        // $dailyLog = DailyLog::find($id);
        DailyLog::destroy($id);
    }

```

- #### Vue View

```jsx
<script setup>
import Layout from "../../Layouts/Layout.vue";
import { Link, useForm } from "@inertiajs/inertia-vue3";
import { Inertia } from "@inertiajs/inertia";

defineProps({
	activityLogs: Object
})

const form = useForm();

function deleteData(id) {
	form.delete(route('activity.destroy', id));
}



</script>

<template>
	<Layout>
		<h1>Activity Log</h1>
		<section>
			<div class="activity-log-container">
				<table class="table table-responsive">
					<thead class="thead-light">
						<tr>
							<th>#</th>
							<th>Date</th>
							<th>Time</th>
							<th>Accomplishment</th>
							<th>Remarks</th>
							<th>Option</th>
						</tr>
					</thead>
					<tbody>
						<tr v-for="activity in activityLogs">
							<td>{{ activity.id }}</td>
							<td>{{ activity.date }}</td>
							<td>{{ activity.time }}</td>
							<td>{{ activity.accomplishment }}</td>
							<td>{{ activity.remarks }}</td>
							<td>
								<Link :href="route('activity.edit', activity.id)"
									class="d-inline-block btn btn-primary m-2">Edit</Link>
								<form @submit.prevent="deleteData(activity.id)">
									<button class="d-inline-block btn btn-primary m-2">Delete</button>
								</form>
							</td>
						</tr>
					</tbody>
				</table>

			</div>
		</section>
	</Layout>

</template>
```

##### Full Controller Source Code Example

```php
<?php

namespace App\Http\Controllers;


use App\Models\DailyLog;
use Illuminate\Http\Request;

class ActivityController extends Controller
{

    public function index()
    {
        $activityLogs = DailyLog::all();
        return inertia("Activity/ActivityLog", [
            "activityLogs" => $activityLogs
        ]);
    }


    public function create()
    {
        return inertia("Activity/ActivityCreateForm");
    }

    public function store(Request $request)
    {
        //
        $postData = $request->all();
        DailyLog::create($postData);
        return redirect()->route("activity.index");
    }
    public function show($id)
    {
        return redirect()->route('activity.index');
    }


    public function edit($id)
    {
        return inertia("Activity/ActivityEditForm", [
            "data" => DailyLog::find($id)
        ]);
    }


    public function update(Request $request, $id)
    {
        $dailyLog = DailyLog::find($id);

        $dailyLog->date = $request->date;
        $dailyLog->time = $request->time;
        $dailyLog->accomplishment = $request->accomplishment;
        $dailyLog->remarks = $request->remarks;
        $dailyLog->update();

        return redirect()->route('activity.index');
    }

    public function destroy($id)
    {
        // $dailyLog = DailyLog::find($id);
        DailyLog::destroy($id);
    }
}
```
