
# how to fix laravel use CASHIER

## After Installing laravel 6 and  CASHIER 10

Edit `.env` and set your Stripe keys:
if your user Model is in "App\User"
add CASHIER_MODEL=App\Models\User

```
STRIPE_KEY=
STRIPE_SECRET=
CASHIER_MODEL=
```


## Note

i search all over the internet to use laravel 6 and CASHIER 10 
i didnt find so i make a solution to use  CASHIER 10 in laravel



### have 2 solution:
*if u whant a multi payment methods u need to add a table
and use function store()
else u can use the fast why: 
FastStore() without use Stripe Package more.


```
        public function FastStore(){
            
            $user = $request->user();
            
             //without Stripe Package
            
            if( $user->stripe_id == null ){
                $user->createAsStripeCustomer(['source' => $request->get('token')]);
                $user->updateDefaultPaymentMethodFromStripe();
            }
            
            if($user->hasPaymentMethod()){
                 //can make a  
                //newSubscription('plan_name',stripe(.com)->products->Pricing-plans->Details->id)
                $user->newSubscription('planname', 'Pricing-plans-id')
                    ->create($user->paymentMethods()[0]->id);
            }
            
            
        }
        
```


```
   
        public function store(){

          try {
                // need to use 
                // use Stripe\Stripe; //insall Stripe Package
                
                Stripe::setApiKey(config('services.stripe.secret'));
          
                /*
                *   if App/User is in App\Models\User dont forgert in env.file to add:
                *   CASHIER_MODEL=App\Models\User
                */
                $user = $request->user();

                if( $user->stripe_id == null ){
                    $user->createAsStripeCustomer();
                }

                //create  PaymentMethod (need Stripe Package)
                //use Stripe\PaymentMethod as StripePaymentMethod;
                $paymentMethod = StripePaymentMethod::create([
                    'type' => 'card',
                    'card' => [
                        'token' => $request->get('token'),
                      ],
                ]);


                //update the user table with the paymentMethod we create
                $user->updateDefaultPaymentMethod($paymentMethod);

                //can make a  
                //newSubscription('plan_name',stripe(.com)->products->Pricing-plans->Details->id)
                $user->newSubscription('planname', 'Pricing-plans-id')->create($paymentMethod);


                return back()->with('status', 'Successful');
                //return response()->json('Successful', 200);

            } catch (\Exception $ex) {
                return response()->json($ex->getMessage(), 200);
            }

```    
