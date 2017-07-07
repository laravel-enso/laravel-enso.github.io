tinker

fisierele trebuie copiate in storage/app/

stergem .idea, .git

$files = array_filter(Storage::allFiles('pics'), function($pic) {return strpos($pic, '.png') !== false;});

foreach($files as $file) {$name = str_replace('.png', '_thumb.png', $file); \Storage::copy($file, $name);}

$files = array_filter(Storage::allFiles('pics'), function($pic) {return strpos($pic, '_thumb.png') !== false;});

foreach ($files as $file) {$image = \Image::make(storage_path('app/'.$file)); $image->resize(847.5, null, function ($constraint) {$constraint->aspectRatio();}); $image->save(storage_path('app/'.$file));}

$optimizer = (new ImageOptimizer\OptimizerFactory)->get();

foreach ($files as $file) {$optimizer->optimize(storage_path('app/'.$file));}